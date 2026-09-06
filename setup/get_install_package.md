# Installation & Setup Guide

This document outlines how package management and system bootstrapping are structured within this dotfiles repository.

## Overview

Bootstrapping logic and instal system package installation scripts are stored seprately from user binaries in the `setup/` directory.

---

## Package Installation

The package installer script autmatically detects your Linux distribution and installs the corresponding list of packages.

### Script: `setup/install-packages.sh`

Supported distributions and target files:

| Distribution Family                   | Detection Key (`ID`)                   | Target Package File   | Package Manager        |
| ------------------------------------- | -------------------------------------- | --------------------- | ---------------------- |
| **Arch Linux**                        | `arch`                                 | `packages/arch.txt`   | `pacman -Syu --needed` |
| **Debian / ubuntu / Mint / Pop!\_OS** | `debina`, `ubuntu`, `linuxmint`, `pop` | `packages/debian.txt` | `apt_get` install -y`  |

> Any unsupported distribution terminates with an error code and diagnostic message.

### Usage

#### 1. Via Zsh Helper Function (Recommended)

once the dotfiles enviroment is sourced, invoke the package installer using:

```zsh
dotinstall
```

This helper function is declared in `common/.config/zsh/modules/dotfiles.zsh`

```bash
dotinstall() {
    "$DOTFILES/setup/install-packages.sh"
}
```

#### 2. Direct Execution

You can also invoke the script directly from your terminal:

```bash
chmod +x "$HOME/.dotfiles/setup/install-packages.sh"
"$HOME/.dotfiles/setup/install-packages.sh"
```
