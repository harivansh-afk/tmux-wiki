# tmux Configuration and Option Scopes

> Sources: tmux Project, Unknown; Nicholas Marriott, Unknown
> Raw: [README](../../raw/tmux/README); [example_tmux.conf](../../raw/tmux/example_tmux.conf); [tmux.1](../../raw/tmux/tmux.1); [options-table.c](../../raw/tmux/options-table.c)
> Updated: 2026-04-23

## Overview

tmux configuration is not a separate declarative format. A config file is a sequence of tmux commands that runs when the server first starts, and later changes use the same command surface through `source-file`, the prompt, or the CLI. Options are organized by scope with explicit inheritance, so understanding configuration means understanding where an option lives and which broader scope supplies its fallback value.

## Configuration Loading

The README points users to `tmux.1` and the example configuration file rather than a distinct config language reference. The man page says tmux first loads the system configuration, then a user configuration file, when the server starts. Those commands are executed once for that server lifetime, and later reloads happen through `source-file`.

This means changes to `.tmux.conf` do not automatically affect an already running server. The mental model is "run tmux commands at startup" rather than "keep a file continuously synchronized."

## Option Scopes and Inheritance

The man page divides options into server, session, window, and pane scopes. Server options are global. Sessions inherit from global session options. Windows inherit from global window options. Panes inherit from window options and then, if needed, from global window defaults. User options are separate string values whose names start with `@`.

`options-table.c` is the master definition of built-in options. It records each option's visible type, allowed values, default, and scope. That file is the most authoritative place to understand whether an option is a string, number, flag, choice, colour, key, or command, and whether it belongs to server, session, window, or pane state.

## Changing Options

`set-option` and `show-options` are the core tools for managing configuration at runtime. The man page highlights a few important behaviors:

- `-s`, `-w`, and `-p` choose server, window, or pane scope.
- `-g` changes global session or window defaults.
- `-a` appends to string or style options instead of replacing them.
- `-u` and `-U` unset options so inherited values take effect again.
- Flag and choice options can be toggled when the value is omitted.

This gives tmux configuration a compositional feel. You often start from defaults and layer changes on top rather than rewriting a full configuration object.

## Practical Patterns from the Example Config

`example_tmux.conf` demonstrates the style of configuration tmux expects:

- status-line customization with `set -g`
- terminal capability tweaks with `terminal-features`
- prefix remapping by changing `prefix`, unbinding `C-b`, and binding `send-prefix`
- mouse behavior changes through options plus explicit unbinds
- session and window bootstrap commands such as `new`, `neww`, and `setw`

This file is useful as a pattern library because it mixes appearance, key bindings, window creation, and conditional logic in the same command language.

## See Also

- [tmux Command Parsing and Key Bindings](command-parsing-and-key-bindings.md)
- [tmux Session, Window, and Pane Model](session-window-and-pane-model.md)
- [tmux Architecture and Process Model](architecture-and-process-model.md)
