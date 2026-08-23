# Dotfiles Documentation

Central documentation hub for dotfiles configuration modules, developer tools, and workflow automation scripts.

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Quick Command Reference](#-quick-command-reference)
- [Scripts & Tools](#-scripts--tools)
  - [Lab (Environment Manager)](#lab-environment-manager)
  - [Focus Mode](#focus-mode)
  - [Tmux Project Manager (TPM)](#tmux-project-manager-tpm)
  - [Package Installer](#package-installer)
- [Configuration Modules](#-configuration-modules)
  - [Zsh Modules](#zsh-modules)
- [Related Repositories](#-related-repositories)

---

## 🧭 Overview

This repository contains usage guides, architecture notes, and reference manuals for the utilities packaged within the [dotfiles](https://github.com/sync-config/dotfiles.git) ecosystem.

---

## ⚡ Quick Command Reference

| Command | Purpose                                               | Documentation                                                                                      |
| :------ | :---------------------------------------------------- | :------------------------------------------------------------------------------------------------- |
| `lab`   | Ephemeral & persistent Git worktree dashboard (`fzf`) | [Lab Docs](https://github.com/sync-config/dotfiles-docs/blob/main/scripts/lab/user_guide.md)       |
| `focus` | Distraction-free workspace / audio focus trigger      | [Focus Docs](https://github.com/sync-config/dotfiles-docs/blob/main/scripts/focus.md)              |
| `tpm`   | Interactive tmux session & project workspace manager  | [TPM Docs](https://github.com/sync-config/dotfiles-docs/blob/main/scripts/tmux_project_manager.md) |

---

## 🛠 Scripts & Tools

### Lab (Environment Manager)

An interactive terminal tool for managing persistent, isolated Git workspaces using `git worktree` and `fzf`.

```bash
# Launch interactive dashboard
lab
```

- **Features:** Search-or-create workspaces, dirty/clean status indicators, safe deletion (`Ctrl-D`).
- **Full Guide:** See [Lab Documentation](https://github.com/sync-config/dotfiles-docs/blob/main/scripts/lab/user_guide.md) and [Architecture Internals](https://github.com/sync-config/dotfiles-docs/blob/main/scripts/lab/architecture.md).

---

### Focus Mode

Toggles the ambient audio stream and minimal workspace profile for deep-work sessions.

```bash
# Toggle focus environment
focus
```

- **Full Guide:** See [Focus Documentation](https://github.com/sync-config/dotfiles-docs/blob/main/scripts/focus.md)

---

### Tmux Project Manager (TPM)

Quickly attaches to, creates, or switches between tmux sessions across active projects.

```bash
# Open interactive session switcher
tpm
```

- **Full Guide:** See [Tmux Project Manager Documentation](https://github.com/sync-config/dotfiles-docs/blob/main/scripts/tmux_project_manager.md).

---

### Package Installer

Automated package installer and synchronizer for base system dependencies.

- **Full Guide:** See [Installation Guide](scripts/get_install_package.md).

---

## ⚙ Configuration Modules

### Zsh Modules

Detailed documentation regarding shell aliases, exports, functions, and plugin integration:

- [Dotfiles Core Module](https://github.com/sync-config/dotfiles-docs/blob/main/common/dotconfig/zsh/modules/dotfiles.md)

---

## 🔗 Related Repositories

- [dotfiles](https://github.com/sync-config/dotfiles.git) — Main dotfiles repository containing configurations and executables.
