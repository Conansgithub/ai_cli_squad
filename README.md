# AI CLI Squad

A terminal app that manages multiple AI coding agents ([Claude Code](https://github.com/anthropics/claude-code), [Codex](https://github.com/openai/codex), [Gemini](https://github.com/google-gemini/gemini-cli), [Aider](https://github.com/Aider-AI/aider)) in separate workspaces, allowing you to work on multiple tasks simultaneously.

![Screenshot](assets/screenshot.png)

## Highlights

- Complete tasks in the background (including yolo / auto-accept mode!)
- Manage instances and tasks in one terminal window
- Review changes before applying them, checkout changes before pushing them
- Each task gets its own isolated git workspace, so no conflicts
- **[Planned]** Task coordination and progress synchronization between agents

## Installation

### From Source

```bash
git clone https://github.com/Conansgithub/ai_cli_squad.git
cd ai_cli_squad
go build -o cs ./main.go
mv cs ~/.local/bin/  # or anywhere in your PATH
```

### Prerequisites

- [tmux](https://github.com/tmux/tmux/wiki/Installing)
- [gh](https://cli.github.com/) (optional, for GitHub integration)
- Go 1.21+ (for building from source)

## Usage

```bash
# Start the TUI
cs

# Start with auto-accept mode
cs -y

# Use with different AI assistants
cs -p "aider ..."
cs -p "codex"
cs -p "gemini"

# Reset all instances
cs reset

# Show help
cs --help
```

## Keyboard Shortcuts

### Instance Management
| Key | Action |
|-----|--------|
| `n` | Create new session |
| `N` | Create new session with prompt |
| `D` | Delete selected session |
| `↑/j` `↓/k` | Navigate between sessions |

### Actions
| Key | Action |
|-----|--------|
| `Enter/o` | Attach to session |
| `Ctrl+Q` | Detach from session |
| `s` | Commit and push to GitHub |
| `c` | Checkout (pause session) |
| `r` | Resume paused session |
| `?` | Show help |

### Navigation
| Key | Action |
|-----|--------|
| `Tab` | Switch preview/diff tab |
| `Shift+↑/↓` | Scroll in diff view |
| `q` | Quit |

## How It Works

1. **tmux** - Creates isolated terminal sessions for each agent
2. **git worktrees** - Isolates codebases so each session works on its own branch
3. **TUI** - Simple terminal interface for navigation and management

## Roadmap

- [ ] Task coordination system
- [ ] Progress synchronization between agents
- [ ] Agent-to-agent communication
- [ ] GitHub Issues integration
- [ ] Dependency management between tasks

## Project Structure

```
ai_cli_squad/
├── main.go              # CLI entry point
├── app/                 # TUI application layer
├── session/             # Session management
│   ├── instance.go      # Instance lifecycle
│   ├── git/             # Git worktree isolation
│   └── tmux/            # Tmux session management
├── ui/                  # UI components
├── config/              # Configuration
├── daemon/              # Background daemon
└── openspec/            # Specifications
```

## Configuration

Configuration file location: `~/.claude-squad/config.json`

```json
{
  "default_program": "claude",
  "auto_yes": false,
  "branch_prefix": "username/"
}
```

## License

[AGPL-3.0](LICENSE.md)

## Acknowledgments

This project is a derivative work based on [Claude Squad](https://github.com/smtg-ai/claude-squad) by [smtg-ai](https://github.com/smtg-ai).

Original project licensed under AGPL-3.0. This derivative work maintains the same license.
