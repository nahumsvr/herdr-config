# Launchers: shortcuts that open other apps

Two layers launch external tools: **herdr** (keybindings in `config.toml`) and **zsh** (aliases and
a pet widget in `~/.zshrc`, versioned in `zsh/` and `pet/`).

## herdr: popups and tabs

The prefix is `ctrl+b`. In Ghostty, `cmd+a` sends it (see [ghostty-tricks.md](ghostty-tricks.md)).
Press the prefix, release it, then press the second key.

| Shortcut | Opens | Type | Notes |
| --- | --- | --- | --- |
| `prefix`, `ctrl+g` | `lazygit` | popup, 90% x 85% | Git UI. Closes when you quit lazygit. |
| `prefix`, `ctrl+d` | `lazydocker` | popup, 90% x 85% | Containers, logs, stats. |
| `prefix`, `ctrl+r` | `pet exec --color` | popup, 90% x 85% | Snippet picker. Runs the chosen command inside the popup. |
| `prefix`, `shift+c` | `zsh` | popup, 90% x 85% | Floating scratch shell. `exit` closes it. |
| `prefix`, `ctrl+v` | `nvim` | new tab | Creates a tab labelled `nvim` and runs Neovim in it. |
| `ctrl+shift+p` (`cmd+p` in Ghostty) | `herdr-switcher` | popup, 70% x 60% | Jump to an agent, tab or workspace. |

A **popup** is a floating pane that closes when its program exits, so the apps above do not leave
terminals behind. Sizes are set per binding with `width` and `height`.

> `prefix`, `ctrl+v` launches `nvim`, not `lvim`. If you want LunarVim there, change `nvim` to
> `lvim` at the end of that binding's `command` in `config.toml`.

### Adding another launcher

```toml
[[keys.command]]
key = "prefix+ctrl+t"
type = "popup"          # or "shell" to run a command without a popup
command = "btop"
width = "90%"
height = "85%"
description = "btop popup"
```

Then run `herdr server reload-config`. Pick a key that is not already bound.

## herdr: navigation shortcuts that call a plugin

`ctrl+h/j/k/l` are `plugin_action` bindings handled by `vim-herdr-navigation`. If the focused pane
runs Vim or Neovim, the key goes to the editor's split; otherwise herdr moves pane focus.

## zsh: aliases and functions

Defined in the `pet + dev shortcuts` block of `~/.zshrc`, versioned in [`zsh/zshrc`](../zsh/zshrc).
The pet snippets and config are in [`pet/`](../pet).

| Command | Runs |
| --- | --- |
| `lg` | `lazygit` |
| `lzd` | `lazydocker` |
| `v` | `lvim` |
| `dps` / `dpsa` | `docker ps` as a table (running / all) |
| `dlogs <container>` | `docker logs -f --tail 100` |
| `ll` | `eza -l --icons --git` |
| `pex` | `pet exec --color` |
| `pnew` / `pedit` | `pet new` / `pet edit` |
| `j` | Fuzzy-jump to a folder (2 levels under Desktop, Documents, Downloads, CLionProjects, DataGripProjects, `~/.config`) |

I did not use `pp`, `pn` or `ld` because they are existing system binaries.

## pet: snippet manager

- **`ctrl+s`** in the shell opens pet and puts the chosen command on your prompt. It is a zsh
  widget on purpose: a snippet like `cd ~/Downloads` run with `pet exec` happens in a subprocess
  and cannot change your shell's directory, while a command placed on the prompt can.
- **`prefix`, `ctrl+r`** in herdr runs the snippet directly in a popup. Use it for commands that
  do not need to change your shell state (docker, git, ports); `cd:` snippets will not stick there.
- Snippets live in `~/.config/pet/snippet.toml`, with `<param>` and `<param=default>` placeholders
  that pet asks for when you run them. Edit them with `pedit`.

`ctrl+s` normally freezes the terminal (flow control); the `~/.zshrc` block disables that with
`stty -ixon`.

## fzf

`source <(fzf --zsh)` adds `ctrl+r` (history), `ctrl+t` (files) and `alt+c` (cd). `alt+c` does not
work with `macos-option-as-alt = false`, so use `j` instead.

## Requirements

`lazygit`, `lazydocker`, `pet`, `fzf`, `fd`, `eza`, `jq`, and `nvim` or `lvim`, all installable
with Homebrew except `lvim`.
