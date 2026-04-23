# tmux Architecture and Process Model

> Sources: tmux Project, Unknown; Nicholas Marriott, Unknown
> Raw: [README](../../raw/tmux/README); [tmux.1](../../raw/tmux/tmux.1); [tmux.c](../../raw/tmux/tmux.c); [server.c](../../raw/tmux/server.c); [spawn.c](../../raw/tmux/spawn.c); [window.c](../../raw/tmux/window.c)
> Updated: 2026-04-23

## Overview

tmux is built around a long-lived server process that owns the actual terminal state, while user-facing clients attach to that server to display and control sessions. Sessions group windows, windows host panes, and panes are backed by PTYs plus virtual screen state, which is why tmux can survive client disconnects and later reattach without losing the managed terminal world.

## Server and Client Roles

The man page describes tmux as a single server managing all sessions while separate client processes attach to display them. The server and clients communicate over a Unix socket under `/tmp`, and startup options such as `-L` and `-S` select which socket to use. In `tmux.c`, socket paths are expanded from `TMUX_TMPDIR` and validated before the server starts.

`server.c` shows the server lifecycle more concretely. `server_start` may daemonize, reinitialize libevent, create the `server` process wrapper, build key tables, initialize the global trees for sessions and windows, and begin listening on the server socket. This reinforces that tmux's real state lives in the server, not in whichever client happens to be visible.

## Persistent State and Object Graph

The server manages clients, sessions, windows, and panes as separate layers of identity. A session is the user-facing container that survives detach and reconnect. A window is not owned by exactly one session: `window.c` stores windows globally and links them into sessions through `winlink` objects, which is why a single window can appear in multiple sessions. A pane is a PTY-backed terminal region inside a window, with its own buffers and virtual screen representation.

`window.c` makes that layering explicit. Each pane has input and output buffers, receives PTY output that is parsed into screen state, and can later be redrawn for any attached client. The virtual screen abstraction is the reason reattaching to tmux restores a coherent terminal display rather than replaying raw shell output from scratch.

## Process and Pane Creation

`spawn.c` is the bridge between tmux's object model and the actual shell or program running inside a pane. Creating a window or pane means collecting the session history limit, base index, working directory, PATH, default shell, termios, and environment, then building a new pane or replacing an existing one. The code distinguishes between creating a fresh window and respawning into an existing slot, but in both cases the session and window metadata are updated first and the child process is then attached to that managed structure.

This means tmux's "terminal multiplexer" behavior is not just visual. It is a runtime that allocates PTYs, remembers layout and history, and links process execution into named session state.

## Architectural Implications

Because the server owns state, clients can be read-only, detached, resized independently in some cases, or replaced entirely without destroying the session. Because windows are global objects linked into sessions, commands that move, link, or unlink windows are manipulating references rather than copying terminal contents. Because panes hold virtual screen state, higher-level features such as copy mode, redraw, and control-mode output can build on stable internal representations instead of scraping terminal pixels.

## See Also

- [tmux Session, Window, and Pane Model](session-window-and-pane-model.md)
- [tmux Command Parsing and Key Bindings](command-parsing-and-key-bindings.md)
- [tmux Configuration and Option Scopes](configuration-and-option-scopes.md)
- [tmux Control Mode](control-mode.md)
