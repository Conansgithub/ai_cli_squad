# Instance Management Specification

## Overview

实例管理模块负责 AI 代理实例的完整生命周期管理，包括创建、启动、暂停、恢复和删除。

## Requirements

### Requirement: Instance Creation

系统 SHALL 允许用户创建新的 AI 代理实例，每个实例包含独立的 Git Worktree 和 Tmux 会话。

#### Scenario: Create new instance with title
- **WHEN** 用户按 `n` 键并输入实例标题
- **THEN** 系统创建新实例，分配独立的 worktree 和 tmux 会话
- **AND** 实例状态设为 Running

#### Scenario: Instance limit reached
- **WHEN** 已有 10 个实例时用户尝试创建新实例
- **THEN** 系统显示错误信息，拒绝创建

### Requirement: Instance Lifecycle

系统 SHALL 支持实例的完整生命周期：Running → Paused → Resumed → Killed。

#### Scenario: Pause instance
- **WHEN** 用户对运行中的实例按 `c` 键
- **THEN** 系统提交本地变更到分支
- **AND** 删除 worktree（保留分支）
- **AND** 分离 tmux 会话
- **AND** 实例状态设为 Paused

#### Scenario: Resume paused instance
- **WHEN** 用户对暂停的实例按 `r` 键
- **THEN** 系统检查分支未被其他 worktree 占用
- **AND** 重建 worktree
- **AND** 恢复 tmux 会话
- **AND** 实例状态设为 Running

#### Scenario: Kill instance
- **WHEN** 用户对实例按 `D` 键并确认
- **THEN** 系统关闭 tmux 会话
- **AND** 删除 worktree 和分支
- **AND** 从列表中移除实例

### Requirement: Instance Status Detection

系统 SHALL 每 500ms 轮询一次实例状态，检测输出变化和提示符。

#### Scenario: Detect output update
- **WHEN** tmux 会话输出内容变化
- **THEN** 实例状态设为 Running

#### Scenario: Detect prompt waiting
- **WHEN** 输出包含确认提示（如 "Yes/No"）
- **AND** AutoYes 模式开启
- **THEN** 自动发送 Enter 键

### Requirement: Instance Persistence

系统 SHALL 在退出时持久化所有实例状态，重启后自动恢复。

#### Scenario: Save on exit
- **WHEN** 用户退出应用
- **THEN** 所有实例状态序列化到 state.json

#### Scenario: Restore on startup
- **WHEN** 应用启动
- **THEN** 从 state.json 恢复实例列表
- **AND** 非 Paused 状态的实例自动重连 tmux

## Implementation Notes

- 实例数据结构：`session/instance.go` 中的 `Instance` struct
- 状态枚举：`Running`, `Ready`, `Loading`, `Paused`
- 存储位置：`~/.claude-squad/state.json`
