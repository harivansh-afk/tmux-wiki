# tmux Session, Window, and Pane Model

> Sources: tmux Project, Unknown; Nicholas Marriott, Unknown
> Raw: [README](../../raw/tmux/README); [tmux.1](../../raw/tmux/tmux.1); [session.c](../../raw/tmux/session.c); [window.c](../../raw/tmux/window.c); [spawn.c](../../raw/tmux/spawn.c)
> Updated: 2026-04-23

## Overview

tmux exposes sessions, windows, and panes as user-visible concepts, but internally they are distinct layers with different ownership rules. Sessions are named collections with their own options and activity state, windows are globally tracked objects linked into sessions by index, and panes are the individual PTY-backed terminals that actually run processes and store screen history.

## Sessions

The man page defines a session as a collection of pseudo terminals under tmux management. `session.c` shows that a session carries a name, numeric ID, working directory, environment, options, terminal settings, and activity timestamps. Sessions are kept in a global RB tree and remain alive until explicitly destroyed or until reference counts drop to zero after unlinking and detach flows finish.

Sessions are also where some default behaviors originate. `spawn.c` pulls the history limit, base index, working directory, shell, and environment from the session when it creates new windows or panes. So a session is more than a label for a tab group: it is an execution context that shapes later object creation.

## Windows and Window Links

Users often talk about a window as if it belongs to a session, but `window.c` shows a more precise model. Windows live in a global tree and are linked into sessions with `winlink` structures that provide the session-local index. This explains why tmux can link a window into multiple sessions and why commands such as `link-window`, `move-window`, and `unlink-window` are fundamentally about membership and indexing.

When a new window is created, `spawn_window` chooses an index, creates a `winlink`, allocates the underlying window if needed, and then links the two together. The `base-index` option matters because default numbering is computed from session options rather than being hard-coded.

## Panes

Each pane is a separate terminal region and a separate PTY. The man page notes that panes are numbered from zero in creation order and can be split horizontally or vertically, resized, swapped, rotated, or selected independently. `window.c` adds the implementation detail that each pane has buffers and a virtual screen, so pane operations are manipulating real terminal state rather than a purely geometric layout.

This is also why pane-local features work the way they do. The active pane, marked pane, copy mode state, and pane-specific options are attached to persistent pane identities, not just to coordinates on screen.

## Modes and Buffers

The window-and-pane layer is where tmux stops being only a shell launcher and starts acting like an interface environment. The man page distinguishes direct terminal access from special modes:

- Copy mode reads pane history and writes selections into paste buffers.
- View mode reuses much of the same machinery for command output.
- Choose mode presents selectable lists of buffers, clients, sessions, windows, or panes.

These modes sit on top of the pane's saved screen and history, which is why they can work even when the underlying application is not cooperating.

## See Also

- [tmux Architecture and Process Model](architecture-and-process-model.md)
- [tmux Command Parsing and Key Bindings](command-parsing-and-key-bindings.md)
- [tmux Configuration and Option Scopes](configuration-and-option-scopes.md)
