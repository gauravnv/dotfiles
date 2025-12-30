# dotfiles

This is my dotfile setup for my Pop!_OS linux machine Zephyrus G14. It started in Dec 2025 and I'm sure it will evolve over time. If you happen to visit and see some room for improvement, please feel free to open a PR, issue, or just reach out. Would be happy to learn! :)

## What’s Included

- **Shell**: Zsh + Oh My Zsh + Powerlevel10k
- **Terminal**: Kitty (Dracula theme)
- **CLI UX**: `eza`, `zoxide`, `fzf`, `ripgrep`, `bat`, `fd`, `gh`, `tmux`, `direnv`, `uv`
- **Desktop**: COSMIC shortcut definitions
- **VS Code**: `settings.json` + extension list
- **System info**: Fastfetch config + a sample image logo

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

It:
- Shows `chezmoi diff` (machine vs repo)
- Optionally runs `chezmoi re-add` (pull local changes into the repo)
- Shows repo diffs (uses `delta` if installed)
- Optionally commits
- Optionally pushes directly or opens a PR via `gh`

## Highlights

- `ff` → `fastfetch`
- `shortcuts` → shortcut reference output
- `fzp` → `fzf` with `bat` preview
- `tn` / `ta` / `tl` → tmux new/attach/list
- `uvr` / `uvs` / `uva` → `uv run` / `uv sync` / `uv add`

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
