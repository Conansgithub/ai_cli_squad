# UI Components Specification

## Overview

UI 组件模块基于 Bubble Tea 框架，提供终端用户界面，包括实例列表、预览窗格、Diff 显示和菜单栏。

## Requirements

### Requirement: Instance List

系统 SHALL 显示所有实例的列表，支持导航和选择。

#### Scenario: Display instance list
- **WHEN** 应用启动
- **THEN** 显示所有实例列表
- **AND** 每个条目显示：状态图标、标题、仓库名（可选）、Diff 统计

#### Scenario: Navigate list
- **WHEN** 用户按 `↑`/`↓` 或 `j`/`k`
- **THEN** 移动选择高亮
- **AND** 更新预览窗格显示选中实例的输出

### Requirement: Preview Pane

系统 SHALL 显示选中实例的终端输出预览。

#### Scenario: Display preview
- **WHEN** 选中实例
- **THEN** 显示该实例的 tmux 输出
- **AND** 每 100ms 自动刷新

#### Scenario: Scroll preview
- **WHEN** 用户在预览窗格中滚动
- **THEN** 支持上下滚动查看历史输出

### Requirement: Diff Pane

系统 SHALL 显示选中实例的 Git 变更。

#### Scenario: Display diff
- **WHEN** 切换到 Diff 标签
- **THEN** 显示添加/删除行数统计
- **AND** 显示彩色 Diff 内容（绿色添加、红色删除）

### Requirement: Tabbed Window

系统 SHALL 支持预览和 Diff 之间切换。

#### Scenario: Switch tabs
- **WHEN** 用户按 Tab 键
- **THEN** 在 Preview 和 Diff 之间切换

### Requirement: Menu Bar

系统 SHALL 显示当前可用的快捷键菜单。

#### Scenario: Context-aware menu
- **WHEN** 选中运行中的实例
- **THEN** 显示：n/N(new), Enter(attach), s(submit), c(checkout), D(delete)
- **WHEN** 选中暂停的实例
- **THEN** 显示：r(resume), D(delete)

### Requirement: Overlay Dialogs

系统 SHALL 支持弹出式对话框。

#### Scenario: Confirmation dialog
- **WHEN** 执行危险操作（删除、推送）
- **THEN** 显示确认对话框
- **AND** 等待用户确认或取消

#### Scenario: Text input dialog
- **WHEN** 创建新实例
- **THEN** 显示文本输入框
- **AND** 支持字符输入、删除、确认

### Requirement: Error Display

系统 SHALL 在底部显示错误信息。

#### Scenario: Show error
- **WHEN** 发生错误
- **THEN** 在底部显示红色错误信息
- **AND** 3 秒后自动消失

## State Machine

```
┌─────────────────────────────────────────────────┐
│                 stateDefault                     │
│  正常浏览模式，显示列表和预览                      │
└─────────────────────────────────────────────────┘
       │           │            │           │
    按 n/N      按 Enter      按 ?        按 c/D
       │           │            │           │
       ▼           ▼            ▼           ▼
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│ stateNew │ │ Attach   │ │stateHelp │ │stateConf │
│ 输入标题  │ │ 进入会话  │ │ 显示帮助  │ │ 确认操作  │
└──────────┘ └──────────┘ └──────────┘ └──────────┘
       │                        │           │
    按 Enter                  按 q/Esc    按 y/n
       │                        │           │
       ▼                        ▼           ▼
┌──────────┐             ┌─────────────────────┐
│statePromp│             │    stateDefault     │
│ 输入提示词│             │                     │
└──────────┘             └─────────────────────┘
```

## Implementation Notes

- 框架：Bubble Tea (charmbracelet/bubbletea)
- 样式：Lip Gloss (charmbracelet/lipgloss)
- 核心文件：`ui/` 目录下的所有组件
- 状态管理：`app/app.go` 中的 `home` struct
