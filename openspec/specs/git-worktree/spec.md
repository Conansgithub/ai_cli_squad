# Git Worktree Isolation Specification

## Overview

Git Worktree 模块负责为每个实例创建独立的工作目录，实现代码隔离，避免多个 AI 代理同时修改同一文件。

## Requirements

### Requirement: Worktree Creation

系统 SHALL 为每个实例创建独立的 Git Worktree，使用唯一的分支名和目录路径。

#### Scenario: Create worktree for new instance
- **WHEN** 创建新实例
- **THEN** 在 `~/.claude-squad/worktrees/` 下创建新目录
- **AND** 目录名格式为 `<branch>_<timestamp>`
- **AND** 创建新分支 `<prefix>/<instance-title>`
- **AND** 执行 `git worktree add -b <branch> <path> HEAD`

#### Scenario: Worktree with existing branch
- **WHEN** 分支已存在（如恢复暂停的实例）
- **THEN** 执行 `git worktree add <path> <branch>`
- **AND** 不创建新分支

### Requirement: Branch Naming

系统 SHALL 使用配置的前缀和实例标题生成分支名。

#### Scenario: Generate branch name
- **GIVEN** 配置前缀为 `alice/`
- **AND** 实例标题为 `fix-auth-bug`
- **THEN** 分支名为 `alice/fix-auth-bug`

### Requirement: Worktree Removal

系统 SHALL 在实例暂停或删除时正确清理 worktree。

#### Scenario: Remove worktree on pause
- **WHEN** 实例暂停
- **THEN** 执行 `git worktree remove -f <path>`
- **AND** 保留分支（不删除）

#### Scenario: Cleanup worktree on kill
- **WHEN** 实例删除
- **THEN** 执行 `git worktree remove -f <path>`
- **AND** 删除对应分支
- **AND** 执行 `git worktree prune` 清理孤立数据

### Requirement: Conflict Prevention

系统 SHALL 防止同一分支被多个 worktree 检出。

#### Scenario: Prevent duplicate checkout
- **WHEN** 尝试恢复实例
- **AND** 分支已被其他 worktree 检出
- **THEN** 返回错误，拒绝恢复

### Requirement: Diff Statistics

系统 SHALL 提供 worktree 的变更统计（添加/删除行数）。

#### Scenario: Calculate diff stats
- **WHEN** 请求实例的 diff 统计
- **THEN** 执行 `git diff --stat`
- **AND** 返回添加行数、删除行数和完整 diff 内容

## Implementation Notes

- 核心文件：`session/git/worktree.go`, `worktree_ops.go`
- Worktree 目录：`~/.claude-squad/worktrees/`
- 共享对象库：所有 worktree 共享 `.git/objects`，节省磁盘空间
