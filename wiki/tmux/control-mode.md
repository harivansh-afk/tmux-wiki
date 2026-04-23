# tmux Control Mode

> Sources: tmux Project, Unknown; Nicholas Marriott and George Nachman, Unknown
> Raw: [tmux.1](../../raw/tmux/tmux.1); [control.c](../../raw/tmux/control.c)
> Updated: 2026-04-23

## Overview

Control mode is tmux's programmatic interface for external tools. Instead of drawing a terminal UI, tmux speaks a line-oriented text protocol over standard input and output. Commands still use normal tmux syntax, but results are wrapped in structured `%begin`/`%end` or `%error` blocks, and asynchronous notifications describe changes in clients, sessions, windows, panes, buffers, and subscriptions.

## Protocol Shape

The man page says a control-mode client sends tmux commands or command sequences terminated by newlines. Each command produces one output block that starts with `%begin`, contains zero or more payload lines, and ends with `%end` or `%error`. The wrapper includes a time value, command number, and flags field.

Notifications are separate from command output and are guaranteed not to appear inside an output block. Examples include `%output`, `%layout-change`, `%window-add`, `%session-changed`, and `%subscription-changed`.

## Internal Queueing and Flow Control

`control.c` reveals that tmux implements control mode with explicit buffering and per-pane tracking. Each client has one queue of all control blocks plus pane-specific queues. `%output` blocks are added to both so ordering can be preserved even when a client is slow to read. Non-output notifications stay in the all-client queue until earlier pane output has been flushed.

The file also makes tmux's backpressure strategy concrete:

- `CONTROL_BUFFER_LOW` is 512 bytes.
- `CONTROL_BUFFER_HIGH` is 8192 bytes.
- `CONTROL_WRITE_MINIMUM` is 32 bytes.
- `CONTROL_MAXIMUM_AGE` is 300000.

These are implementation details, but they matter because they explain why pause-related flags exist and why control mode is designed around bounded buffering rather than unbounded streaming.

## Pause and Subscription Semantics

The man page documents `pause-after` and extended output notifications for cases where tmux deliberately pauses a control client that is falling behind. `control.c` shows matching internal state for pending panes, paused flags, and subscription tracking. Subscriptions are keyed by name and format and cache the last emitted value, which means tmux can avoid emitting unchanged subscription payloads.

For tooling, the key lesson is that control mode is not just "run tmux commands and scrape text." It is a stateful event stream with explicit flow-control behavior.

## See Also

- [tmux Architecture and Process Model](architecture-and-process-model.md)
- [tmux Command Parsing and Key Bindings](command-parsing-and-key-bindings.md)
