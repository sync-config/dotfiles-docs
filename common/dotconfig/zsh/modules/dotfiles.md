---
# Dotfiles Setup Guide

A complete reference for deploying your dotfiles to a new machine and managing changes across systems.
---

## 1. Prerequisites

On a new system, only these tools must be installed before starting:

| Tool   | Purpose              | Install              |
| ------ | -------------------- | -------------------- |
| `git`  | Fetch the repository | from package manager |
| `stow` | Create symlinks      | from package manager |

On **Arch**:

```bash
sudo pacman -S git stow
```

On **Debian/Ubuntu**:

```bash
sudo apt-get update
sudo apt-get install -y git stow
```

> `zsh` must be installed separately if you want it, but it's not required just to deploy the config.

---

## 2. Clone the repository

```bash
git clone <REPO_URL> ~/.dotfiles
```

For example:

```bash
git clone https://github.com/USERNAME/dotfiles.git ~/.dotfiles
```

Or via SSH:

```bash
git clone git@github.com:USERNAME/dotfiles.git ~/.dotfiles
```

---

## 3. Install base packages (optional but recommended)

```bash
bash ~/.dotfiles/scripts/install-packages.sh
```

This script detects the distro (Arch or Debian) and installs the appropriate packages.

> If the script is not yet executable:
>
> ```bash
> chmod +x ~/.dotfiles/scripts/install-packages.sh
> ```

---

## 4. Apply config with Stow

### First time (with `--adopt`)

If the new system already has default configs (e.g. `.zshrc` or `~/.config/nvim` already exists), use `--adopt`:

```bash
stow --dir="$HOME/.dotfiles" --target="$HOME" --adopt common
```

This:

- Moves existing files into the repository (without destroying them)
- Creates symlinks

Then check the repository status:

```bash
git -C ~/.dotfiles status
git -C ~/.dotfiles diff
```

If there are unwanted changes:

```bash
git -C ~/.dotfiles checkout -- .
```

### If the system is clean (no prior config)

```bash
stow --dir="$HOME/.dotfiles" --target="$HOME" --restow common
```

---

## 5. Define aliases and functions

Add these once to your shell. For zsh, put them in `~/.zshrc`; for bash, in `~/.bashrc`.

```bash
export DOTFILES="$HOME/.dotfiles"

alias dotfiles='git -C "$DOTFILES"'
alias dotstatus='git -C "$DOTFILES" status'
alias dotdiff='git -C "$DOTFILES" diff'
alias dotlog='git -C "$DOTFILES" log --oneline --decorate -10'
```

```bash
dotapply() {
    stow --dir="$DOTFILES" --target="$HOME" --restow common
}

dotinstall() {
    "$DOTFILES/scripts/install-packages.sh"
}

dotsync() {
    git -C "$DOTFILES" pull --ff-only || return 1
    dotapply
}

dotsave() {
    local message="${*:-dotfiles: update}"
    git -C "$DOTFILES" add -A &&
        git -C "$DOTFILES" commit -m "$message" &&
        git -C "$DOTFILES" push
}
```

Reload your shell afterwards:

```bash
source ~/.zshrc   # or ~/.bashrc
```

> These aliases can also live inside your dotfiles (in `common/.config/shell/` or directly in `.zshrc`), so they become active automatically after `dotapply`.

---

## 6. Final verification

```bash
ls -l ~/.zshrc
ls -l ~/.config/nvim/init.lua
ls -l ~/.tmux.conf
```

The output should show a `->` pointing to paths inside `~/.dotfiles/`:

```text
~/.zshrc -> /home/USER/.dotfiles/common/.zshrc
```

---

# Day-to-day workflow

## Making a change on one system

1. Edit the file directly (it's a symlink):

```bash
nvim ~/.config/nvim/init.lua
```

2. Save and push:

```bash
dotsave "nvim: fix telescope keymap"
```

This command:

- `add -A` → stages all changes
- `commit` → commits with the given message
- `push` → pushes to GitHub

## Pulling changes on another system

```bash
dotsync
```

This:

- `pull`s (fast-forward only)
- Re-applies symlinks (`dotapply`)

## Adding a new dependency

Suppose a new Neovim plugin requires `ripgrep`:

1. Install it on the current system:

```bash
sudo pacman -S ripgrep    # Arch
# or
sudo apt-get install -y ripgrep    # Debian
```

2. Add it to the manifests:

```bash
# packages/arch.txt
ripgrep

# packages/debian.txt
ripgrep
```

3. Save:

```bash
dotsave "packages: add ripgrep"
```

4. Apply on the other system:

```bash
dotsync
dotinstall
```

---

# Command summary

| Command         | Action                                |
| --------------- | ------------------------------------- |
| `dotsave "msg"` | Save and push config changes          |
| `dotsync`       | Pull latest config and apply symlinks |
| `dotinstall`    | Install shared packages               |
| `dotapply`      | Apply symlinks only (no pull)         |
| `dotstatus`     | Show change status                    |
| `dotdiff`       | Show diff                             |
| `dotlog`        | Show commit history                   |

---

# Important notes

## 1. `--adopt` only once

Use `--adopt` only for the **initial** deployment. In day-to-day use, `dotapply` runs without `--adopt`, because:

- `--adopt` may overwrite the repo file with the system file
- After symlinks are created, edits happen directly on the repo file, so adopting is unnecessary

## 2. Neovim plugins

After `dotsync` on a new system, sync the plugins:

```bash
nvim --headless "+Lazy! sync" +qa
```

Or inside Neovim:

```vim
:Lazy sync
```

The `lazy-lock.json` file must be committed to Git to keep plugin versions identical across systems.

## 3. Machine-specific files

If some config differs between systems (e.g. a project-specific path), don't put it in the repo. Instead:

- Keep it in a separate file (e.g. `~/.config/local.env`)
- Or use an environment variable
- Or add it to `.gitignore`

## 4. Security

Never commit secret files (passwords, tokens, SSH keys) to the repo. Use `.gitignore`:

```text
# .gitignore in ~/.dotfiles/
.env
*.secret
**/.env.local
```

---

# Troubleshooting

## `cannot stow ... over existing target` error

A real file already exists at that path. Fix:

```bash
stow --dir="$HOME/.dotfiles" --target="$HOME" --adopt common
git -C ~/.dotfiles status
git -C ~/.dotfiles diff
```

Revert if needed:

```bash
git -C ~/.dotfiles checkout -- .
```

## Symlinks not created

Check Stow is installed:

```bash
command -v stow
```

If not:

```bash
sudo pacman -S stow       # Arch
sudo apt-get install stow # Debian
```

## `dotsync` gives a merge conflict

Local and remote commits have diverged. Check the status:

```bash
dotstatus
```

If you only have local changes and want to accept the remote:

```bash
git -C "$DOTFILES" fetch
git -C "$DOTFILES" reset --hard origin/main   # or origin/master
dotapply
```

> ⚠️ This discards local changes; only do it when you're sure the remote is correct.

---

You can save this directly as `~/.dotfiles/README.md` so it travels with the repo.

If you'd like, I can also generate this as a clean PDF or Markdown file for you.
