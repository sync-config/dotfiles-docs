# Tmux Project Manager (TPM)

![Tmux Project Manager Preview](../demo/tpm.gif)

A lightweight terminal utility for managing project-based `tmux` sessions with an interactive [`fzf`](https://github.com/junegunn/fzf) interface and dynamic workspace directory resolution.

The script allows you to:

- Search through existing `tmux` sessions and workspace directories
- Attach to a session from outside `tmux`
- Switch to another session from inside `tmux`
- Interactively configure and persist your root workspace directory (`PROJECT_DIR`)
- Automaticaly create a new project directory and `tmux` session when no matching session exists
- Delete a selected `tmux` session directly from the `fzf` interface

---

## Features

- Interactive project/session search using `fzf`
- Dynamic environment configuration: prompts for workspace directory on first launch
- Automatic detection of the current `tmux` context (`attach` vs `switch-client`)
- Automatic creation of project directories under `$PROJECT_DIR`
- Automatic creation of new `tmux` sessions
- Session preview inside `fzf`
- Session deletion directly from the picker using `Ctrl+D`
- Current session hidden from the session list to avoid recursive attachment or accidental deletion
- Support for inline runtime environment overrides

---

## Requirements

The following tools must be installed and available in your `PATH`.

### tmux

The script uses `tmux` to create, switch, attach to, and delete terminal sessions.

- [tmux documentation](https://github.com/tmux/tmux)

### fzf

The script uses `fzf` to provide an interactive fuzzy-search interface.

- [fzf repository](https://github.com/junegunn/fzf)

### Installation

#### Arch Linux

```bash
sudo pacman -S tmux fzf
```

#### Debian/Ubuntu

```bash
sudo apt install tmux fzf
```

---

##‌ Configuration & Enviroment Management
TPM relies on the `PROJECT_DIR` variable to locate your projects and scaffold new repositories.
**First-Run Setup**
if `PROJECT_DIR` is not yet configured, the script automatically triggers an interactive first-run setup:

```text
[TPM] PROJECT_DIR is not set.
Please enter your base projects directory (e.g. ~/Projects or /home/user/workspace):
> ~/Projects
[TPM] Saved PROJECT_DIR to /lib/project_manager/.env
```

onece entered, the directory is validated, resolved, and saved.

**Configuration File (`lib/project_manager/.env`)**

- **File Location:** lib/project_manager/.env
- **Git-lgnored & Machine-Specific:** This file is intentionally trached by `.gitignore` Your project path on your Arch workstation can differ from your laptop or server without creating dirty Git state.
- **Content Format:** Standard POSIX key-value pairs:

```bash
PROJECT_DIR="/home/user/Projects"
```

**Supported Path Formats**
The setup script cleanly handles both relative shorthand and absolute paths:

- **Tilde Expansion:** `~/Projects`, `~/work/repos` (expanded to `$HOME/...`)
- **Absolute Paths:** `/home/user/Projects`, `/mnt/data/workspace`

## Usage

Run the project manager command via alias or direct binary:

```bash
# Using alias/symlink
tpm

# Or binary directly
tmux-project-manager
```

**Manual Overrides**
you can override your default porject directory at runtime without modifying your persisted configuration:

1. **Runtime lnline Overrride (Temporary)**

Run TPM targeting a different directory for a single execution:

```bash
PROJECT_DIR=~/work/client-b tpm
```

2. **Manual Configuration Update (Persistent)**

To chenge your default workspace directory permanently, edit `lib/project_manager/.env`

```bash
# Open with your editor of choice
nvim lib/project_manager/.env
```

Update the PROJECT_DIR value:

```bash
PROJECT_DIR="/home/user/NewWorkspace"
```

---

## How It Works

### 1. Search for a session or Directory

When the script starts, it list active `tmux` session alongside subdirectories in your `$PROJECT_DIR`.

Type part of a session or project name to filter the list:

```text
Project> dot
```

Select the desired entry and press `Enter`.

### 2. Switch to a session from inside tmux

If TPM is executed inside an active `tmux` window, it switches context using:

```bash
tmux switch-client -t <session-name>
```

### 3. Attach to a session from outside tmux

If executed from a standalone terminal shell, TPM attaches via:

```bash
tmux attach-session -t <session-name>
```

### 4. Create a New Project and Session

If you type a name that does not exist:

1. TPM creates `$PROJECT_DIR/<new-project-name>`.
2. It initializes a new session named `<new-project-name>` rooted in that folder.
3. It immediately switches to or attaches the new session.

For example, entering `api-gateway` creates:

```text
$PROJECT_DIR/api-gateway
```

and spawns a session named `api-gateway`

---

## Keybindings

| Key      | Action                                               |
| -------- | ---------------------------------------------------- |
| `Enter`  | Attach to or switch to the selected session          |
| `Ctrl+D` | Delete the selected `tmux` session                   |
| `Esc`    | Exit without selecting a session                     |
| Typing   | Filter existing sessions or enter a new project name |

> **warning:** Ctrl+D sends a kill command to the selected session.All running processes, panes, and buffers inside that session will be terminated immediatedly.

---

## Session Preview

The `fzf` preview pane displays live diagnostics for the highlighted session:

- Session name & creation date
- Active working directory
- Window and pane breakdown
- Recent terminal scrollback/output from the active pane

---

## Troubleshooting & Reset

### Resetting configuration

To re-trigger the interactive first-run prompt, simply remove the local `.env` file:

```bash
rm -f lib/project_manager/.env
```

Next time you launch `tpm`, it will prompt for the directory setup again.

### Verifying Environment Resolution

Verify what TPM is reading by inspecting the configuration file:

```bash
cat lib/project_manager/.env
```

Make sure the directory actually exists:

```bash
# Substiture your path path or read directly
eval $(cat lib/project_manager/.env)
ls -la "$PROJECT_DIR"
```

### Dependencies Check

```bash
tmux -V
fzf --version
```

### Check Active tmux Sessions

```bash
tmux list-sessions
```

---

## Safety Notes

- Session Deletion: Ctrl+D permanently kills the target session. The active session you are currently working in is excluded from the picker list to protect against self-termination.
- Directory Sanitization: Project directory names should avoid whitespace and non-POSIX shell characters (/, \, quotes) to prevent path resolution and session creation issues.
