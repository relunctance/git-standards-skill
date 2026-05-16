# git-standards-skill
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)]()
[![category](https://img.shields.io/badge/category-devops-blue.svg)]()

[![license](https://img.shields.io/badge/license-MIT-blue.svg)](#)
[![platforms](https://img.shields.io/badge/platforms-claude_code%20%7C%20openclaw%20%7C%20hermes-blue.svg)](#)
[![version](https://img.shields.io/badge/version-1.0.0-green.svg)](#)

标准化 Git commit、push、工作流的 skill，支持代理自动切换，强制所有改动即时 commit + push。

## 触发条件

当遇到以下场景时自动调用此 skill：

| 场景 | 说明 |
|------|------|
| git commit 规范 | 不知道如何写符合规范的 commit message |
| git push 失败 | 网络超时、认证失败、代理问题 |
| GitHub 代理 | push 时连接不上 GitHub |
| 分支策略 | 不知道如何命名/操作分支 |
| git 改动未提交 | 有本地改动需要 commit 但不知道规范 |

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

### 规则

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

### push 失败自动处理

```
git push → 失败 → 检测错误类型 → 修复 → 重试
```

| 错误类型 | 自动修复 |
|---------|---------|
| timeout / Connection refused | 自动查找代理并配置 |
| authentication failed | 提示用 `gh auth login` |
| remote 不存在 | 提示检查 remote URL |

## 分支命名

```
<type>/<short-description>

示例：
feat/user-auth
fix/lance-error
docs/api-design
chore/update-deps
```

规则：全小写，用 `-` 分隔，简洁 ≤ 3 个词。

## 安装

```bash
# Hermes 全局
mkdir -p ~/.hermes/skills/git-standards-skill
ln -sf ~/repos/git-standards-skill/SKILL.md ~/.hermes/skills/git-standards-skill/SKILL.md

# Hermes baijie agent
mkdir -p ~/.hermes/profiles/baijie/skills/git-standards-skill
ln -sf ~/repos/git-standards-skill/SKILL.md ~/.hermes/profiles/baijie/skills/git-standards-skill/SKILL.md

# Claude Code
mkdir -p ~/claude/skills/git-standards-skill
ln -sf ~/repos/git-standards-skill/SKILL.md ~/claude/skills/git-standards-skill/SKILL.md
```

## 配合其他 skills

| 场景 | 调用 skill |
|------|-----------|
| push 失败需要诊断/代理 | `github-push-solver-skill` |
| commit message 需要优化 | `darwin-skill`（可选） |
| 新项目初始化 repo | `skill-created` |

## 踩坑

| 坑 | 原因 | 解决 |
|------|------|------|
| push 后还有未 push 的 commit | 多个独立改动没分开 commit | 每次只 commit 相关文件 |
| commit 描述写了代码行数 | 混淆了 what 和 why | 描述用户价值或技术原因 |
| 强制 push 覆盖了别人的代码 | 没有用 `--force-with-lease` | 用 `git push --force-with-lease` |
| .env 提交后无法从 git 删除 | 只删了文件没从缓存删 | `git rm --cached .env` |
| 本地 commit 堆积 | 习惯最后一起 push | **立即 commit + push** |
