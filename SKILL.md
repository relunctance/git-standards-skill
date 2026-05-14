---
name: git-standards-skill
description: 当需要 git commit 规范、push 失败处理、GitHub 代理配置、分支策略时使用。标准化 commit message 格式、push 失败自动修复代理、分支命名规范，强制所有改动即时 commit + push
version: "1.0.0"
author: relunctance
license: MIT
category: devops
tags:
  - git
  - github
  - commit
  - push
  - devops
  - standards
metadata:
  hermes:
    platforms:
      claude_code: true
      openclaw: true
      hermes: true
    related_skills: [github-push-solver-skill, skill-created]
---

# git-standards-skill

GQL 团队的 Git 工作流规范 — 标准化 commit、push、branch 策略，支持代理自动切换，所有改动强制即时提交。

## 核心原则

> **每完成一个独立任务单元，立即 commit + push，不堆积。**

## Commit 规范

### 格式

```
<type>: <简短描述>

[可选的详细说明]
```

### Type 列表

| Type | 适用场景 | 示例 |
|------|---------|------|
| `feat` | 新功能 | `feat: 新增用户登录功能` |
| `fix` | Bug 修复 | `fix: 修复 search 空数组报错` |
| `docs` | 文档更新 | `docs: 更新 README 安装步骤` |
| `style` | 代码格式（不影响功能） | `style: format with gofmt` |
| `refactor` | 重构（不修 bug 不加功能） | `refactor: 拆分 config.go` |
| `perf` | 性能优化 | `perf: 优化 recall 延迟` |
| `test` | 测试相关 | `test: 新增 recall benchmark` |
| `chore` | 杂项（依赖、构建、工具） | `chore: update go.mod` |
| `ci` | CI/CD 配置 | `ci: add github actions` |
| `revert` | 回滚 | `revert: revert abc1234` |

### Commit 规则

1. **描述 ≤ 72 字符**，用中文，简洁明确
2. **Body** 描述 **why** 不描述 **what**（what 是代码本身）
3. **每条 commit 是原子操作** — 一个 commit 只做一件事
4. **禁止无意义 commit**：不要 "update"、"fix typo"、"asdf" 这种
5. **不要 commit 未完成的工作** — 用 `git stash` 暂存

### 好 vs 坏的 commit

```
✅ feat: 新增 mflow client 集成 recall 流程
✅ fix: 修复 LanceDB 向量更新后 stale 的问题
✅ docs: 更新 hawk-bridge v2 架构文档

❌ update                          # 太模糊
❌ fix typo                        # 没有价值
❌ WIP                             # 未完成不 commit
❌ asdf                            # 无意义
❌ 新增功能（修复了之前的问题）  # 一条 commit 做了两件事
```

## Push 规范

### 必须 push 的时机

**每次 commit 后立即 push**，不允许本地有未 push 的 commit。

### push 失败处理流程

```
git push → 失败 → 检测错误类型 → 修复 → 重试
```

| 错误类型 | 自动修复 |
|---------|---------|
| timeout / Connection refused | 自动查找代理并配置 |
| authentication failed | 提示用 gh auth login |
| remote 不存在 | 提示检查 remote URL |

### 代理自动配置

push 失败时自动执行，**优先从 `.bashrc` 读取已有代理配置**：

```bash
# 1. 从 ~/.bashrc 读取现有代理配置
detect_bashrc_proxy() {
    # 匹配格式：export http_proxy=... / export HTTP_PROXY=... / export ALL_PROXY=...
    local proxy
    proxy=$(grep -E '^export\s+(http_proxy|HTTP_PROXY|https_proxy|HTTPS_PROXY|ALL_PROXY)=' ~/.bashrc 2>/dev/null | head -1 | cut -d'=' -f2 | tr -d '"' | tr -d "'")
    if [ -n "$proxy" ]; then
        echo "$proxy"
        return 0
    fi
    return 1
}

# 2. 若 .bashrc 无配置，扫描常见端口
scan_proxy_ports() {
    for port in 7890 7891 1080 8118 10808; do
        if curl -s --connect-timeout 2 -x http://127.0.0.1:$port https://www.google.com > /dev/null 2>&1; then
            echo "http://127.0.0.1:$port"
            return 0
        fi
    done
    return 1
}

# 3. 应用代理
PROXY=$(detect_bashrc_proxy) || PROXY=$(scan_proxy_ports)
if [ -n "$PROXY" ]; then
    git config --global http.proxy "$PROXY"
    git config --global https.proxy "$PROXY"
    echo "✅ 代理已配置: $PROXY"
else
    echo "⚠️  未找到可用代理，请手动配置"
fi
```

### GitHub Token 自动配置

push 失败认证问题时，检查 `.bashrc` 是否有 `gh_token` 或 `GITHUB_TOKEN`：

```bash
# 从 ~/.bashrc 读取 GitHub Token
detect_gh_token() {
    grep -E '^export\s+(gh_token|GITHUB_TOKEN|GH_TOKEN)=' ~/.bashrc 2>/dev/null | head -1 | cut -d'=' -f2 | tr -d '"' | tr -d "'"
}

TOKEN=$(detect_gh_token)
if [ -n "$TOKEN" ]; then
    # 写入 git credential helper
    git config --global credential.helper store
    echo "https://${TOKEN}@github.com" > ~/.git-credentials
    echo "✅ GitHub Token 已配置"
else
    echo "⚠️  未找到 gh_token，请运行: gh auth login"
fi
```

## 分支策略

### 分支命名

```
<type>/<short-description>

示例：
feat/user-auth
fix/lance-error
docs/api-design
chore/update-deps
```

**规则**：
- 全小写，用 `-` 分隔，不用 `_`
- 简洁，≤ 3 个词描述
- 类型用 `feat`/`fix`/`docs`/`chore` 等

### 分支操作

```bash
# 创建并切换
git switch -c feat/new-feature

# 推送（首次）
git push -u origin HEAD

# 合并后删除本地
git branch -d feat/new-feature

# 合并后删除远程
git push origin --delete feat/new-feature
```

## 工作流

### 标准 Git 工作流

```
1. git status          # 查看改动
2. git diff            # 确认改动内容
3. git add <文件>      # 分批 add（相关文件一起 commit）
4. git commit -m ""    # 按规范 commit
5. git push            # 立即 push
6. 验证 push 成功      # 确认没有遗留未 push 的 commit
```

### 每日开始

```bash
cd ~/repos/<project>
git status
git log --oneline -3
# 检查是否有未 push 的 commit
git cherry -v origin/main
```

### Commit 前自检清单

- [ ] 改动是否完成了？（不 commit 未完成的工作）
- [ ] 描述是否 ≤ 72 字符？
- [ ] 描述是否用中文、简洁明确？
- [ ] 是否只做了一件事？（原子性）
- [ ] 是否有不该提交的文件？（node_modules、.env、*.log）

## 忽略不该提交的文件

### 全局忽略模板

```gitignore
# 依赖
node_modules/
vendor/
.env

# 构建产物
dist/
build/
*.o
*.a

# IDE
.vscode/
.idea/
*.swp
*.swo

# 系统
.DS_Store
Thumbs.db

# 日志
*.log
npm-debug.log*

# 测试覆盖率
coverage/
*.cover

# 本地配置
.env.local
.env.*.local
config.local.json

# 语言特定
__pycache__/
*.pyc
*.pyo
*.so
*.dylib
```

### 已在 .gitignore 但仍有问题的文件

```bash
# 从 git 缓存移除（但保留本地文件）
git rm -r --cached <文件或目录>
git rm -r --cached .
git add .
```

## 常见场景操作

### 场景 1：修复 bug 和加新功能同时发现

```bash
# 先 stash 新功能
git stash push -m "WIP: new feature"
git commit -m "fix: 修复 bug"
git push

# 恢复新功能继续工作
git stash pop
```

### 场景 2：commit 后发现还有相关改动需要提交

```bash
# 先检查当前状态
git status

# 如果是同一个功能，补到上一个 commit
git add <遗漏的文件>
git commit --amend --no-edit

# 如果是独立功能，新建 commit
git add <文件>
git commit -m "feat: 补充相关改动"
git push
```

### 场景 3：push 失败需要代理

```bash
# 自动执行（参考 github-push-solver）
# 1. 测试 GitHub 连通性
curl -s --connect-timeout 5 https://github.com > /dev/null 2>&1

# 2. 若不通，查找代理
for port in 7890 7891 1080 8118 10808; do
    if curl -s --connect-timeout 2 -x http://127.0.0.1:$port https://www.google.com > /dev/null 2>&1; then
        git config --global http.proxy "http://127.0.0.1:$port"
        git config --global https.proxy "http://127.0.0.1:$port"
        echo "✅ 代理已配置: http://127.0.0.1:$port"
        break
    fi
done

# 3. 重试 push
git push
```

### 场景 4：撤销未 push 的 commit

```bash
# 撤销上一次 commit，保留改动
git reset --soft HEAD~1

# 撤销上一次 commit，不保留改动（危险）
git reset --hard HEAD~1
```

### 场景 5：撤销已 push 的 commit

```bash
# 先在本地撤销
git reset --hard <目标commit>

# 强制 push（需要确认安全）
git push --force-with-lease origin HEAD
```

### 场景 6：查看某个 commit 的改动

```bash
git show <commit-hash>          # 完整展示
git show <commit-hash> --stat  # 仅统计
git diff <hash1>..<hash2>      # 两 commit 差异
```

### 场景 7：stash 多个改动

```bash
git stash list                  # 列出所有 stash
git stash push -m "描述"       # 暂存当前改动
git stash pop                   # 恢复最新 stash
git stash pop stash@{n}        # 恢复指定 stash
```

## Git Status 解读

| 状态 | 含义 |
|------|------|
| `Changes not staged for commit` | 文件有改动，未 add |
| `Changes to be committed` | 已 add，待 commit |
| `Your branch is ahead of 'origin'` | 有 commit 未 push |
| `nothing to commit, working tree clean` | 全部已 commit + push |

## 快速参考

```bash
# 查看当前状态
git status --short

# 查看未 push 的 commits
git log origin/main..HEAD --oneline

# 查看远程追踪关系
git branch -vv

# 强制重置到远程
git fetch origin
git reset --hard origin/main

# 查看 config
git config --list --local
git config --list --global
```

## 踩坑

| 坑 | 原因 | 解决 |
|------|------|------|
| push 后还有未 push 的 commit | 多个独立改动没分开 commit | 每次只 commit 相关文件 |
| commit 描述写了代码行数 | 混淆了 what 和 why | 描述用户价值或技术原因 |
| 强制 push 覆盖了别人的代码 | 没有用 `--force-with-lease` | 用 `git push --force-with-lease` |
| .env 提交后无法从 git 删除 | 只删了文件没从缓存删 | `git rm --cached .env` |
| 本地 commit 堆积 | 习惯最后一起 push | **立即 commit + push**，不复查 |
| rebase 后 force push | 多人协作的分支 | 确认分支无其他人工作再用 `--force-with-lease` |

## 与其他 skill 配合

| 场景 | 调用 skill |
|------|----------|
| push 失败需要诊断/代理 | `github-push-solver` |
| commit message 需要优化 | `darwin-skill`（可选） |
| 新项目初始化 repo | `skill-created` |

## 安装

```bash
# Hermes 全局
mkdir -p ~/.hermes/skills/git-standards
ln -sf ~/repos/git-standards/SKILL.md ~/.hermes/skills/git-standards/SKILL.md

# Hermes baijie agent
mkdir -p ~/.hermes/profiles/baijie/skills/git-standards
ln -sf ~/repos/git-standards/SKILL.md ~/.hermes/profiles/baijie/skills/git-standards/SKILL.md

# Claude Code
mkdir -p ~/claude/skills/git-standards
ln -sf ~/repos/git-standards/SKILL.md ~/claude/skills/git-standards/SKILL.md

# Codex
mkdir -p ~/.codex/skills/git-standards
ln -sf ~/repos/git-standards/SKILL.md ~/.codex/skills/git-standards/SKILL.md
```
