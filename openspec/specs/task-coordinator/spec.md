# Task Coordinator Specification

## Overview

任务协调模块（计划中）负责多 AI 代理之间的任务分配、进度同步和状态共享。

**状态**: 🚧 计划中，尚未实现

## Requirements

### Requirement: Task Definition

系统 SHALL 支持创建和管理结构化任务。

#### Scenario: Create task
- **WHEN** 用户创建新任务
- **THEN** 任务包含：ID、标题、描述、状态、分配者、依赖列表
- **AND** 任务持久化到存储

#### Scenario: Task status transitions
- **GIVEN** 任务状态包括：Pending, InProgress, Completed, Blocked
- **WHEN** 任务状态变化
- **THEN** 记录时间戳和变更日志

### Requirement: Task Assignment

系统 SHALL 支持将任务分配给特定实例。

#### Scenario: Assign task to instance
- **WHEN** 将任务分配给实例
- **THEN** 任务的 assigned_to 字段设为实例标题
- **AND** 实例可以查询自己的任务列表

#### Scenario: Unassigned task pool
- **WHEN** 任务未分配
- **THEN** 任务出现在公共待办池中
- **AND** 任何实例可以认领

### Requirement: Dependency Management

系统 SHALL 支持任务之间的依赖关系。

#### Scenario: Define task dependency
- **WHEN** 任务 B 依赖任务 A
- **THEN** 任务 B 状态为 Blocked 直到 A 完成

#### Scenario: Dependency resolution
- **WHEN** 依赖的任务完成
- **THEN** 被阻塞的任务自动变为 Pending

### Requirement: Progress Synchronization

系统 SHALL 支持跨实例的进度同步。

#### Scenario: Update task progress
- **WHEN** 实例更新任务进度
- **THEN** 变更写入共享存储
- **AND** 其他实例可以读取最新状态

#### Scenario: Conflict resolution
- **WHEN** 多个实例同时更新同一任务
- **THEN** 使用最后写入胜出（Last Write Wins）策略
- **OR** 使用乐观锁（版本号检查）

### Requirement: Shared State Storage

系统 SHALL 提供共享状态存储机制。

#### Scenario: File-based storage
- **WHEN** 使用文件存储
- **THEN** 任务数据保存在 `~/.claude-squad/tasks.json`
- **AND** 使用文件锁防止并发写入

#### Scenario: External storage integration
- **WHEN** 集成外部存储（如 GitHub Issues）
- **THEN** 通过 API 同步任务状态
- **AND** 支持双向同步

## Data Model

```go
type Task struct {
    ID          string     `json:"id"`
    Title       string     `json:"title"`
    Description string     `json:"description"`
    Status      TaskStatus `json:"status"`
    AssignedTo  string     `json:"assigned_to"`
    DependsOn   []string   `json:"depends_on"`
    CreatedAt   time.Time  `json:"created_at"`
    UpdatedAt   time.Time  `json:"updated_at"`
}

type TaskStatus string

const (
    TaskPending    TaskStatus = "pending"
    TaskInProgress TaskStatus = "in_progress"
    TaskCompleted  TaskStatus = "completed"
    TaskBlocked    TaskStatus = "blocked"
)

type TaskStore struct {
    Tasks    []Task    `json:"tasks"`
    Version  int       `json:"version"`
    LastSync time.Time `json:"last_sync"`
}
```

## Implementation Notes

- 计划目录：`session/coordinator/`
- 存储位置：`~/.claude-squad/tasks.json`
- 与现有 Instance 集成：通过 instance.Title 关联
