# DNS Management CLI (`dns`)

A lightweight, modular CLI tool designed to inspect, configure, and protect system DNS settings (`/etc/resolv.conf`) against automatic overwrites by network managers (such as NetworkManager, systemd-resolved, or DHCP clients).

## Preview

![dns](../demo/dns.gif)

---

## Table of Contents

- [Overview & Architecture](#overview--architecture)
- [Prerequisites & Dependencies](#prerequisites--dependencies)
- [Installation & System Integration](#installation--system-integration)
- [Permission Model](#permission-model)
- [Commands & Usage Reference](#commands--usage-reference)
  - [`dns status`](#dns-status)
  - [`dns shecan`](#dns-shecan)
  - [`dns lock`](#dns-lock)
  - [`dns unlock`](#dns-unlock)
  - [`dns help`](#dns-help)
- [Troubleshooting & Notes](#troubleshooting--notes)

---

## Overview & Architecture

The `dns` tool unifies generic DNS configuration and Shecan anti-sanction setup under a single entry point (`bin/dns`).

### Directory Layout

```text
bin/
└── dns                 # Main executable / CLI dispatcher
lib/
└── dns/                # Modular internal logic
    ├── status.sh       # Display status /etc/resolv.conf file
    ├── shecan.sh       # Shecan nameserver configuration
    ├── common.sh       # Check Dependencies for run script
    └── lock.sh         # Lock and Unlock /etc/resolv.conf file
```

## Native Immutability Handling

Unlike naive implementations that rely on custom PID/lockfiles, dns uses native Linux filesystem attributes:

- **Locking:** Applies the `+i` (immutable) attribute to `/etc/resolv.conf` via chattr `+i`
- **Unlocking:** Removes the `+i` attribute via chattr `-i`.
- **State Check:** Inspects the current attribute state using lsattr.

This prevents any process (even root/system daemons) from modifying or replacing `/etc/resolv.conf` until explicitly unlocked.

## Prerequisites & Dependencies

- e2fsprogs: Required for `chattr` and lsattr utilities.
  - Arch Linux: `sudo pacman -S e2fsprogs`
  - Debian / Ubuntu: `sudo apt install e2fsprogs`
- sudo / Root Access: Required for mutating operations (shecan, lock, unlock).

## Installation & System Integration

The binary is automatically symlinked into /usr/local/bin during dotfile deployment:

```bash
dotapply
```

This runs the internal `link_binaries` routine, exposing dns directly in your `$PATH`.

## Permission Model

| Action         | Command             | Sudo Required? | Notes                                           |
| -------------- | ------------------- | -------------- | ----------------------------------------------- |
| Inspect status | `dns status / dns ` | ❌No           | Read-only inspection via lsattr & file reading. |
| View help      | `dns help / -h`     | ❌ No          | Read-only CLI help text.                        |
| Apply Shecan   | `sudo dns shecan`   | ✅ Yes         | Modifies `/etc/resolv.conf` and sets `+i`.      |
| Lock file      | `sudo dns lock`     | ✅ Yes         | Runs `chattr +i /etc/resolv.conf`.              |
| Unlock file    | `sudo dns unlock`   | ✅ Yes         | Runs `chattr -i /etc/resolv.conf`.              |

## Commands & Usage Reference

`dns status`
Displays active nameservers from /etc/resolv.conf and checks whether the file is currently immutable.

```bash
dns status
```

```text

[✓] Status: Locked (Immutable)
[i] Current Nameservers:
- 178.22.122.100
- 185.51.200.2
```

`dns shecan`
Temporarily unlocks `/etc/resolv.conf`, writes the Shecan DNS addresses, and locks the file again with `+i`.

- **Nameserver 1:** 178.22.122.100
- **Nameserver 2:** 185.51.200.2

```bash
sudo dns shecan
```

`dns lock`
Sets the immutable bit (`+i`) on `/etc/resolv.conf` to prevent external services from overwriting current DNS entries.

```bash
sudo dns lock
```

`dns unlock`
Removes the immutable bit (`-i`) on `/etc/resolv.conf`, allowing package managers, network managers, or manual edits to update the file.

```bash
sudo dns unlock
```

`dns help`
Prints available subcommands and flags.

```bash
dns help
# or

dns -h
dns --help
```

## Troubleshooting & Notes

Symlinked resolv.conf: If `/etc/resolv.conf` is a symlink (e.g. pointing to systemd-resolved), `chattr` might fail on some filesystems. Ensure `/etc/resolv.conf` is a regular file if you want to use the immutable attribute reliably:

```bash
sudo rm /etc/resolv.conf
sudo touch /etc/resolv.conf
sudo dns shecan
```

Read-Only Filesystems: `chattr` operations require a filesystem that supports extended attributes (e.g., ext4, Btrfs, XFS).
