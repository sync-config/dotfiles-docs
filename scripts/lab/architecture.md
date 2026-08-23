# Lab Architecture

This document describes the internal architecture of `lab`, including its dashboard flow, persistence model, shell isolation, and preview behavior.

---

## Overview

`lab` is built around `git worktree` and an `fzf` dashboard.
It stores persistent session data under `~/.lab/` and uses metadata files to track Labs across sessions.

---

## Main Components

- `~/.dotfiles/scripts/lab`
- `~/.dotfiles/scripts/lab-preview`
- `~/.dotfiles/lib/lab/config.sh`
- `~/.dotfiles/lib/lab/ui.sh`
- `~/.dotfiles/lib/lab/args.sh`
- `~/.dotfiles/lib/lab/git.sh`
- `~/.dotfiles/lib/lab/session.sh`
- `~/.dotfiles/lib/lab/manager.sh`

---

## Storage Layout

Labs are stored under:

```text
~/.lab/
```

Each Lab contains:

- a Git worktree
- `.lab-meta`
- repository files

The `.lab-meta` file is an internal implementation detail and should be treated as opaque by users.

---

## Dashboard Model

The dashboard is driven by `fzf` using:

- `--print-query`
- `--expect='enter,ctrl-d'`

This allows `lab` to distinguish between:

- typed query
- selected item
- pressed action key

---

## Search-or-Create Flow

If the user types a name and presses `Enter` without selecting a matching item, `lab` creates a new Lab.

If an existing Lab is selected and `Enter` is pressed, `lab` opens that session.

---

## Delete Flow

When `Ctrl-D` is pressed on a selected Lab:

1. the selected row is parsed
2. the first tab-separated field is extracted as `session_dir`
3. `lab_delete_session "$session_dir"` is called

Important:

- the full selected row must not be passed directly
- only the first field is valid as the path

---

## Shell Isolation

`LAB_ACTIVE` is only set inside the Lab shell session.
This prevents nested Lab creation and avoids polluting the dashboard process.

`enter_lab_shell` runs in isolated environment context so the dashboard can safely return after the shell exits.

---

## Preview Utility

`lab-preview` is a standalone read-only program used by `fzf`.

It must not source the main `lab` entrypoint because that can cause:

- recursive shell startup
- `SHLVL` warnings
- side effects in interactive preview mode

---

## Debugging

Use:

```bash
LAB_DEBUG=1
```

Tracing should be written using `BASH_XTRACEFD` so debug output does not interfere with the `fzf` interface.

---

## Invariants

The following rules must remain true:

- preview must stay read-only
- delete must use `session_dir`, not the full selected row
- nested Lab sessions must be blocked
- the dashboard must not leak environment state
- invalid or metadata-less directories must be skipped by `lab_list`

---

## Extension Notes

When extending the tool:

- keep UI code separate from filesystem logic
- avoid coupling preview logic to the main shell entrypoint
- preserve backward compatibility of metadata parsing
- test dashboard behavior with both existing Labs and empty queries

---

## Related Files

- [User Guide](https://github.com/sync-config/dotfiles-docs/blob/main/scripts/lab/user_guide.md)
- [README](https://github.com/sync-config/dotfiles-docs/blob/main/README.md)

```

```
