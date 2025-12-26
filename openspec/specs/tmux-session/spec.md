# Tmux Session Management Specification

## Overview

Tmux 会话模块负责在后台运行 AI 代理程序，提供终端输出捕获、键盘输入注入和会话附着/分离功能。

## Requirements

### Requirement: Session Creation

系统 SHALL 为每个实例创建独立的 tmux 会话，运行指定的 AI 程序。

#### Scenario: Create tmux session
- **WHEN** 启动新实例
- **THEN** 执行 `tmux new-session -d -s claudesquad_<name> -c <workdir> <program>`
- **AND** 设置 history-limit 为 10000
- **AND** 启用鼠标支持

#### Scenario: Handle trust prompt
- **WHEN** Claude Code 显示 "Do you trust the files in this folder?"
- **THEN** 自动发送 Enter 键确认

### Requirement: Output Capture

系统 SHALL 实时捕获 tmux 会话的终端输出。

#### Scenario: Capture pane content
- **WHEN** 请求预览内容
- **THEN** 执行 `tmux capture-pane -p -e -J -t <name>`
- **AND** 返回带 ANSI 颜色的内容

#### Scenario: Detect content change
- **WHEN** 轮询会话输出
- **THEN** 计算内容的 SHA256 哈希
- **AND** 与前一个哈希比对
- **AND** 返回是否有更新

### Requirement: Input Injection

系统 SHALL 支持向 tmux 会话注入键盘输入。

#### Scenario: Send prompt to session
- **WHEN** 用户提交提示词
- **THEN** 通过 PTY 写入内容
- **AND** 发送 Enter 键

#### Scenario: AutoYes mode
- **WHEN** 检测到确认提示
- **AND** AutoYes 模式开启
- **THEN** 自动发送 Enter 键

### Requirement: Session Attach/Detach

系统 SHALL 支持交互式附着到 tmux 会话，并安全分离。

#### Scenario: Attach to session
- **WHEN** 用户按 Enter 或 `o` 键
- **THEN** 附着到 tmux 会话
- **AND** 转发所有键盘输入
- **AND** 显示实时输出

#### Scenario: Detach from session
- **WHEN** 用户按 Ctrl+Q
- **THEN** 安全分离会话
- **AND** 返回到列表界面
- **AND** 会话继续后台运行

### Requirement: Session Naming

系统 SHALL 使用统一前缀命名 tmux 会话，避免与用户会话冲突。

#### Scenario: Generate session name
- **GIVEN** 实例标题为 `fix-auth-bug`
- **THEN** tmux 会话名为 `claudesquad_fix-auth-bug`

## Implementation Notes

- 核心文件：`session/tmux/tmux.go`
- 会话前缀：`claudesquad_`
- 平台支持：Unix (`tmux_unix.go`)、Windows (`tmux_windows.go`)
- PTY 管理：通过 `pty.go` 处理伪终端
