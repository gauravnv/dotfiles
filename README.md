# dotfiles

This is my dotfile setup for my Pop!_OS linux machine Zephyrus G14. It started in Dec 2025 and I'm sure it will evolve over time. If you happen to visit and see some room for improvement, please feel free to open a PR, issue, or just reach out. Would be happy to learn! :)

![Fastfetch Sample](/docs/sample.png)

## What’s Included

- **Shell**: Zsh + Oh My Zsh + Powerlevel10k
- **Terminal**: Kitty (Dracula theme)
- **CLI Tools**: Modern replacements
  - `eza` (ls), `zoxide` (cd), `fzf` (fuzzy finder), `ripgrep` (grep), `bat` (cat)
  - `fd` (find), `btop` (top), `ncdu` (disk usage), `tldr` (man pages)
  - `gh` (GitHub CLI), `tmux`, `direnv`, `uv` (Python), `delta` (git diff)
- **Desktop**: COSMIC shortcut definitions
- **VS Code**: `settings.json` + extension list
- **System info**: Fastfetch config + sample image logo
- **Code Quality**: EditorConfig, pre-commit hooks (Biome, Ruff, shellcheck)
- **Documentation**: Agent instructions for AI assistants (AGENTS.md)

## Quick Start

### Install & Apply (New Machine)

```bash
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply gauravnv/dotfiles
```

This repo uses chezmoi run scripts to:
- Install baseline apt packages listed in `packages-apt.txt`
- Install `uv`
- Install Oh My Zsh + Powerlevel10k
- Install VS Code extensions listed in `vscode-extensions.txt`
- Install `delta` via `cargo` (best-effort; skipped if `cargo` is missing)

### Update Workflow

Run the helper script:

```bash
update-dotfiles
```

Common modes:

```bash
update-dotfiles --check
update-dotfiles -y --pull --apply --re-add --commit --push
update-dotfiles --commit --pr
```

It:
- Shows remote ahead/behind (and can fast-forward pull)
- Shows `chezmoi diff` (machine vs repo)
- Optionally runs `chezmoi re-add` (pull local changes into the repo)
- Shows repo diffs (uses `delta` if installed)
- Optionally commits
- Optionally pushes directly or opens a PR via `gh`

## Highlights

### Quick Navigation
- `z <partial-path>` → Jump to frequently visited directory (zoxide)
- `fcd [depth]` → Fuzzy directory picker with preview (default depth: 6)
- `fopen` → Fuzzy file picker with preview
- `fh` → Fuzzy history search

### Modern CLI Tools
- `rg` → ripgrep (smart-case search)
- `fd` → fd-find (faster than find)
- `bat` → batcat (syntax-highlighted cat)
- `eza` (via `ll`, `lt`, `lta`) → Better ls with icons and git status

### Shell Functions
- `mkcd <dir>` → Make directory and cd into it
- `extract <file>` → Extract any archive type
- `gac <msg>` → Git add all + commit
- `gacp <msg>` → Git add all + commit + push
- `gclone <url>` → Clone repo and cd into it
- `psg <name>` → Search processes by name
- `serve [port]` → Quick HTTP server (default: 8000)
- `weather [city]` → Terminal weather forecast
- `cheat <command>` → Quick cheat sheet from cheat.sh

### System Utilities
- `ff` → `fastfetch`
- `shortcuts` → Shortcut reference output
- `reload` → Restart shell
- `path` → Show PATH entries one per line
- `myip` → Show public IP
- `disk` → Interactive disk usage (ncdu)
- `top` → Better system monitor (btop)

### Development
- `tn` / `ta` / `tl` → tmux new/attach/list
- `uvr` / `uvs` / `uva` / `uvp` → `uv run` / `uv sync` / `uv add` / `uv pip`
- Pre-commit hooks configured (Biome, Ruff, shellcheck)

## Fastfetch Logo Image

This repo includes a sample image at `~/.config/fastfetch/images/cat-window.png`.

To use a different image:
1. Put a PNG at `~/.config/fastfetch/images/cat-window.png` (or change the path).
2. Update `~/.config/fastfetch/config.jsonc`.

The path in the fastfetch config is templated using chezmoi (no hardcoded username).

## Hardware Notes (Zephyrus G14)

Hardware-specific notes and fixes (audio/Wi-Fi/asusd/supergfxctl) are documented in:
- `docs/system-setup.md`

Some of these should *not* be auto-applied on other machines.
