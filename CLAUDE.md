<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

# CLAUDE.md - Claude Squad

This file provides guidance to Claude Code when working with this repository.

## 项目概述

Claude Squad 是一个终端应用，用于管理多个 AI 编码代理（Claude Code、Aider、Codex 等）在隔离的工作环境中并行运行。

**核心特性**：
- 多实例管理：最多 10 个并发 AI 代理
- Git Worktree 隔离：每个代理有独立的工作目录
- Tmux 后台运行：支持分离/重连，实例持久化
- TUI 界面：基于 Bubble Tea 的终端用户界面

## 常用命令

### 开发

```bash
# 构建
go build -o cs ./main.go

# 运行
./cs

# 测试
go test ./...

# 代码检查
golangci-lint run
```

### 使用

```bash
# 启动 TUI
cs

# 启动并开启 AutoYes 模式
cs -y

# 重置所有实例
cs reset

# 查看帮助
cs --help
```

### 快捷键

| 按键 | 功能 |
|------|------|
| `n` / `N` | 创建新实例 / 带提示词创建 |
| `↑` `↓` `j` `k` | 导航 |
| `Enter` / `o` | 附着到实例 |
| `Ctrl+Q` | 从实例分离 |
| `s` | 推送变更 |
| `c` | 签出（暂停） |
| `r` | 恢复 |
| `D` | 删除 |
| `Tab` | 切换预览/Diff |
| `?` | 帮助 |
| `q` | 退出 |

## 项目架构

```
claude-squad/
├── main.go              # CLI 入口
├── app/                 # TUI 应用层
│   └── app.go           # 状态机和事件处理
├── session/             # 会话管理层
│   ├── instance.go      # 实例生命周期
│   ├── storage.go       # 持久化
│   ├── git/             # Git Worktree
│   └── tmux/            # Tmux 会话
├── ui/                  # UI 组件
├── config/              # 配置管理
├── daemon/              # 守护进程
├── cmd/                 # 命令执行
├── keys/                # 快捷键定义
├── log/                 # 日志
└── openspec/            # 规格文档
    ├── project.md       # 项目约定
    ├── specs/           # 功能规格
    └── changes/         # 变更提案
```

## 开发规范

### 代码风格

- 遵循 Go 官方风格（gofmt）
- 使用 golangci-lint 检查
- 接口优先，便于测试
- 错误处理使用 defer 确保资源清理

### 提交规范

使用 Conventional Commits：
- `feat:` 新功能
- `fix:` 修复 bug
- `refactor:` 重构
- `docs:` 文档
- `test:` 测试
- `chore:` 杂项

### 测试

- 单元测试使用标准 testing 包
- Mock 通过 `*WithDeps` 构造函数注入
- 集成测试需要真实的 Git 和 Tmux 环境

## 规格文档

详细规格请参考 `openspec/specs/`：

| 规格 | 路径 |
|------|------|
| 实例管理 | `openspec/specs/instance-management/spec.md` |
| Git Worktree | `openspec/specs/git-worktree/spec.md` |
| Tmux 会话 | `openspec/specs/tmux-session/spec.md` |
| UI 组件 | `openspec/specs/ui-components/spec.md` |
| 任务协调（计划中） | `openspec/specs/task-coordinator/spec.md` |

## 计划中的功能

- [ ] 任务协调系统（Task Coordinator）
- [ ] 进度同步和状态共享
- [ ] Agent 间通信机制
- [ ] GitHub Issues 集成
- [ ] 依赖关系管理

## 存储位置

```
~/.claude-squad/
├── config.json      # 配置
├── state.json       # 实例状态
├── daemon.pid       # 守护进程 PID
└── worktrees/       # Git worktree 目录
```

## 许可证

AGPL-3.0 - 修改后必须开源
