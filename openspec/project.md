# Project Context

## Purpose

Claude Squad 是一个终端应用，用于管理多个 AI 编码代理（Claude Code、Aider、Codex 等）在隔离的工作环境中并行运行。

**核心目标**：
- 让开发者能够同时运行多个 AI 代理处理不同任务
- 通过 Git Worktree 实现代码隔离，避免冲突
- 通过 Tmux 实现后台运行和会话持久化
- 提供统一的 TUI 界面管理所有实例

## Tech Stack

- **语言**: Go 1.21+
- **TUI 框架**: Bubble Tea (charmbracelet/bubbletea)
- **CLI 框架**: Cobra
- **终端管理**: Tmux
- **版本控制**: Git Worktree
- **前端（可选）**: Next.js, TypeScript, Tailwind CSS

## Project Conventions

### Code Style

- 遵循 Go 官方代码风格 (gofmt)
- 使用 golangci-lint 进行代码检查
- 接口优先，便于测试和扩展
- 错误处理使用 defer 确保资源清理

### Architecture Patterns

```
分层架构：
├── main.go          → CLI 入口层
├── app/             → 应用层（TUI 状态机）
├── session/         → 会话管理层
│   ├── instance.go  → 实例生命周期
│   ├── git/         → Git Worktree 隔离
│   └── tmux/        → Tmux 会话管理
├── ui/              → UI 组件层
├── config/          → 配置管理层
└── daemon/          → 守护进程层
```

**设计原则**：
- 依赖注入：通过接口抽象（PtyFactory, Executor）便于测试
- 状态机模式：UI 使用 4 种离散状态控制流程
- 资源隔离：每个实例有独立的 worktree 和 tmux 会话

### Testing Strategy

- 单元测试：使用 Go 标准 testing 包
- Mock 注入：通过 `*WithDeps` 构造函数注入测试依赖
- 集成测试：需要真实的 Git 和 Tmux 环境

### Git Workflow

- 分支命名：`<username>/<feature-name>`
- 提交规范：Conventional Commits
- PR 流程：Fork → Feature Branch → PR → Review → Merge

## Domain Context

### 核心概念

| 概念 | 说明 |
|------|------|
| **Instance** | 一个运行中的 AI 代理实例，包含 tmux 会话和 git worktree |
| **Worktree** | Git 的独立工作目录，共享 .git/objects |
| **Session** | Tmux 会话，支持后台运行和 attach/detach |
| **AutoYes** | 自动确认模式，AI 代理自动按 Enter 继续 |

### 实例状态

```
Running  → 正在运行，输出有更新
Ready    → 就绪，等待用户输入
Loading  → 加载中
Paused   → 暂停（保留分支，删除 worktree）
```

### 目录结构

```
~/.claude-squad/
├── config.json      → 配置文件
├── state.json       → 实例状态持久化
├── daemon.pid       → 守护进程 PID
└── worktrees/       → Git worktree 目录
    └── <user>/<task>_<timestamp>/
```

## Important Constraints

1. **实例限制**: 最多 10 个并发实例（GlobalInstanceLimit）
2. **依赖要求**: 必须安装 tmux 和 git
3. **平台支持**: Unix/Linux/macOS 完整支持，Windows 部分支持
4. **许可证**: AGPL-3.0（修改后必须开源）

## External Dependencies

| 依赖 | 用途 | 必需 |
|------|------|------|
| **tmux** | 终端会话管理 | 是 |
| **git** | 版本控制和 worktree | 是 |
| **claude** | Claude Code CLI | 可选（默认程序） |
| **aider** | Aider CLI | 可选 |
| **codex** | OpenAI Codex CLI | 可选 |

## Future Enhancements (计划中)

- [ ] 任务协调系统（Task Coordinator）
- [ ] 进度同步和状态共享
- [ ] Agent 间通信机制
- [ ] GitHub Issues 集成
- [ ] 依赖关系管理
