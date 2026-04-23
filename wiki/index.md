# Knowledge Base Index

## tmux

Core tmux concepts, architecture, and interfaces compiled from the upstream source tree in `raw/tmux/`.

| Article | Summary | Updated |
|---------|---------|---------|
| [tmux Architecture and Process Model](tmux/architecture-and-process-model.md) | Explains how the tmux server, clients, sockets, PTYs, and long-lived state fit together. | 2026-04-23 |
| [tmux Session, Window, and Pane Model](tmux/session-window-and-pane-model.md) | Maps tmux's object model: sessions own window links, windows host panes, and panes back terminal state. | 2026-04-23 |
| [tmux Command Parsing and Key Bindings](tmux/command-parsing-and-key-bindings.md) | Describes how tmux parses commands, queues execution, and maps keys to command lists. | 2026-04-23 |
| [tmux Configuration and Option Scopes](tmux/configuration-and-option-scopes.md) | Covers config loading, option inheritance, and the practical shape of `.tmux.conf`. | 2026-04-23 |
| [tmux Control Mode](tmux/control-mode.md) | Summarizes tmux's text protocol for programmatic clients and the internal flow-control model behind it. | 2026-04-23 |
