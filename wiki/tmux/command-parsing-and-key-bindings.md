# tmux Command Parsing and Key Bindings

> Sources: tmux Project, Unknown; Nicholas Marriott, Unknown
> Raw: [tmux.1](../../raw/tmux/tmux.1); [cmd.c](../../raw/tmux/cmd.c); [key-bindings.c](../../raw/tmux/key-bindings.c)
> Updated: 2026-04-23

## Overview

tmux treats commands as the universal control surface for the program. The same command can be invoked from the shell, from `.tmux.conf`, from the command prompt, or from a key binding. Internally, tmux separates parsing from execution, stores implementations in a central command table, and maps keys to command lists through named key tables.

## Where Commands Come From

The man page calls out four common parsing entry points: configuration files, the command prompt, `bind-key`, and commands such as `if-shell` or `confirm-before` that take other commands as arguments. Shell invocations are first parsed by the shell, while in-tmux entry points are parsed by tmux itself. This is why semicolon escaping and quoting rules differ depending on where the command is written.

Because configuration files are just tmux commands executed in sequence, there is no separate configuration language to learn beyond tmux command syntax.

## Parsing Versus Execution

The `COMMAND PARSING AND EXECUTION` section of `tmux.1` is explicit that tmux first parses a command into a name, flags, and arguments, then later executes it from a command queue. Each client has its own queue, and startup uses a global queue for configuration processing. Some commands insert newly parsed commands back into the queue, while others pause later execution until an external event completes.

This queue model is important for understanding tmux behavior. `if-shell`, `run-shell`, and `display-panes` can delay later commands, so a command sequence is not just a flat list of synchronous calls.

## Command Table and Implementations

`cmd.c` exposes the internal command registry through `cmd_table`, which lists the supported command entries such as `attach-session`, `new-window`, `set-option`, `split-window`, and `wait-for`. Each command has its own implementation file, usually named `cmd-*.c`, and is referenced through a `cmd_entry`.

This organization means tmux's CLI surface is largely data-driven at the top level: parse to a known command entry, then dispatch to that implementation. For wiki maintenance, it also means command-specific behavior is usually easiest to trace by starting from `cmd.c` and then following the corresponding `cmd-<name>.c` file.

## Key Tables and Bindings

`key-bindings.c` stores bindings in named key tables rather than in a single global mapping. A key table owns bindings keyed by internal `key_code` values, and each binding points to a command list. This matches the man page's description of different tables for copy mode, vi-style copy mode, and the default interactive environment.

Two consequences follow from this design:

- Key handling is contextual. The same physical key can do different things in different tables.
- A binding is just another path to command execution. The bound action is not special-case UI code; it is a command list that enters the same execution model as other tmux commands.

The default session, window, and pane menus in `key-bindings.c` show the same pattern: menu actions are emitted as tmux commands rather than handled by a separate menu subsystem API.

## See Also

- [tmux Architecture and Process Model](architecture-and-process-model.md)
- [tmux Configuration and Option Scopes](configuration-and-option-scopes.md)
- [tmux Control Mode](control-mode.md)
