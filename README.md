# herdr-config

My [herdr](https://herdr.dev) configuration: keybindings, theme, sidebar layout and a fuzzy switcher.

## Contents

| Path | Description |
| --- | --- |
| `config.toml` | Main herdr config (theme, sidebar, keybindings, popups) |
| `bin/herdr-switcher` | fzf popup to jump between agents, tabs and workspaces |
| `ghostty/config.ghostty` | Ghostty config, including the `Cmd` -> herdr key forwarding |
| `docs/ghostty-tricks.md` | How the keybinding tricks work |
| `zsh/zshrc` | My `~/.zshrc`, including the pet widget, aliases and `j` jumper |
| `pet/snippet.toml` | pet snippets (docker, compose, git, navigation, network) |
| `pet/config.toml` | pet config (fzf selector, `lvim` editor) |
| `docs/launchers.md` | Shortcuts and aliases that open lazygit, lazydocker, pet, Neovim and more |

## Plugins

| Plugin | Description | Repository |
| --- | --- | --- |
| `dot.terminal-notifier` | macOS notifications (with herdr icon) when an agent is blocked or done | [dot/herdr-terminal-notifier](https://github.com/dot/herdr-terminal-notifier) |
| `vim-herdr-navigation` | Seamless `ctrl+h/j/k/l` navigation between herdr panes and Vim/Neovim splits | [paulbkim-dev/vim-herdr-navigation](https://github.com/paulbkim-dev/vim-herdr-navigation) |

Install them with:

```sh
herdr plugin install dot/herdr-terminal-notifier --yes
herdr plugin install paulbkim-dev/vim-herdr-navigation --yes
```

`dot.terminal-notifier` needs `jq` and a one-time notification permission for **herdr** in
System Settings → Notifications.

## Switcher

`bin/herdr-switcher` lists agents, tabs and workspaces through the herdr CLI and focuses the
selected one. It requires `fzf` and `jq`.

Bound in `config.toml` as a popup on `ctrl+shift+p`. To open it with `cmd+p` from Ghostty,
add this to the Ghostty config (it sends `ctrl+shift+p` as a Kitty CSI-u sequence):

```
keybind = cmd+p=text:\x1b[112;6u
```

## Keybindings

| Key | Action |
| --- | --- |
| `ctrl+t` / `ctrl+shift+w` | New tab / close tab |
| `ctrl+shift+x` | Close pane |
| `ctrl+shift+h` / `ctrl+shift+l` | Previous / next tab |
| `ctrl+shift+j` / `ctrl+shift+k` | Next / previous workspace |
| `ctrl+1..9` | Jump to workspace |
| `ctrl+shift+1..9` | Jump to agent |
| `ctrl+shift+p` | Switcher popup |
| `prefix+ctrl+g` | lazygit popup |
| `prefix+ctrl+d` | lazydocker popup |
| `prefix+ctrl+r` | pet snippets popup |
| `prefix+shift+c` | Floating shell |
| `prefix+ctrl+v` | Neovim in a new tab |

The prefix is `ctrl+b`.

## Launchers

Shortcuts that open other apps (lazygit, lazydocker, pet, a scratch shell, Neovim) and the zsh
aliases around them are documented in [docs/launchers.md](docs/launchers.md).

## Ghostty

`ghostty/config.ghostty` is my Ghostty config. [docs/ghostty-tricks.md](docs/ghostty-tricks.md)
explains how `Cmd` shortcuts are forwarded to herdr and every other keybinding trick used.

## Install

```sh
git clone <this-repo> ~/herdr-config
cp ~/herdr-config/config.toml ~/.config/herdr/config.toml
mkdir -p ~/.config/herdr/bin && cp ~/herdr-config/bin/herdr-switcher ~/.config/herdr/bin/
cp ~/herdr-config/ghostty/config.ghostty "$HOME/Library/Application Support/com.mitchellh.ghostty/config.ghostty"
mkdir -p ~/.config/pet && cp ~/herdr-config/pet/*.toml ~/.config/pet/
herdr server reload-config
```

`zsh/zshrc` is a full copy of my `~/.zshrc` (Oh My Zsh, fnm, PATH entries for my machine). On a new
machine, copy only the `pet + dev shortcuts` block into your own `~/.zshrc` instead of replacing it.

> `pet/config.toml` has empty `access_token` fields. If you ever add a Gist or GitLab sync token to
> pet, remove it before committing.
