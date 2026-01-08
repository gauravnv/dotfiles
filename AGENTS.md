# Agent Instructions

This file provides context for AI assistants (GitHub Copilot, Claude, Cursor, etc.) working in this environment.

## System Context

- **OS**: Pop!_OS (Ubuntu-based Linux) on ASUS ROG Zephyrus G14 (GA403WM)
- **Desktop**: COSMIC (Wayland)
- **Shell**: Zsh with Oh My Zsh + Powerlevel10k
- **Terminal**: Kitty
- **Dotfiles Manager**: chezmoi

## Preferred CLI Tools

Use these modern alternatives instead of legacy commands:

| Instead of | Use | Notes |
|------------|-----|-------|
| `grep` | `rg` (ripgrep) | Faster, respects .gitignore, smart-case by default |
| `find` | `fd` (fdfind on Ubuntu) | Simpler syntax, faster, respects .gitignore |
| `cat` | `bat` (batcat on Ubuntu) | Syntax highlighting, line numbers |
| `ls` | `eza` | Icons, git status, tree view |
| `top`/`htop` | `btop` | Beautiful TUI, resource graphs |
| `du` | `ncdu` | Interactive TUI disk usage analyzer |
| `man` | `tldr` | Simplified examples-first documentation |
| `diff` | `delta` | Syntax-highlighted diffs (configured as git pager) |
| `rm` | `trash-put` | Safe delete to trash instead of permanent |
| `cd` | `z` (zoxide) | Frecency-based smart directory jumping |

## Interactive/TUI Preferences

Prefer interactive and TUI tools when available:

- **Disk usage**: `ncdu` (interactive) over `du`
- **System monitor**: `btop` (TUI) over `top`
- **File selection**: `fzf` for fuzzy finding
- **Git operations**: Use `lazygit` or `gh` CLI when beneficial

## Development Stack

- **Python**: `uv` for package management, `ruff` for linting/formatting
- **JavaScript/TypeScript**: `fnm` for Node version management, `biome` for linting/formatting
- **Containers**: Podman (not Docker)
- **Git hosting**: GitHub, use `gh` CLI

## Code Style

- **Formatting**: Biome (JS/TS), Ruff (Python)
- **Pre-commit**: Configured with biome and ruff hooks
- **EditorConfig**: Spaces, 2-space indent (4 for Python), LF line endings

## File Locations

- **Local scripts**: `~/.local/bin/`
- **Dotfiles source**: `~/gitty/dotfiles/` (chezmoi source)
- **Projects**: `~/gitty/`

## Terminal Commands

When suggesting terminal commands:

1. Use the modern alternatives listed above
2. Prefer pipelines over temporary files
3. Use `fzf` for interactive selection when appropriate
4. Quote variables properly: `"$var"` not `$var`
5. Use `[[ ]]` over `[ ]` for conditionals in zsh/bash

## Examples

```bash
# Finding files (use fd, not find)
fd "\.py$" src/

# Searching content (use rg, not grep)
rg "TODO|FIXME" --type py

# Viewing files (use bat, not cat)
bat src/main.py

# Disk usage (use ncdu for interactive)
ncdu ~/gitty

# Directory navigation (use z, not cd for known paths)
z dotfiles
```
