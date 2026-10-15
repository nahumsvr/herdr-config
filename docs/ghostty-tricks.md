# Ghostty tricks: using Cmd with herdr

This explains how the keybindings in [`ghostty/config.ghostty`](../ghostty/config.ghostty) work,
and why they are needed.

## The core problem: terminals do not see Cmd

A terminal program (herdr, Neovim, zsh) does not receive keys. It receives **bytes**. The
traditional protocol can only encode `Ctrl`, `Alt` and `Shift`. **`Cmd` does not exist in it**: on
macOS it belongs to the app (Ghostty), so `cmd+t` never reaches herdr.

So to use `Cmd` shortcuts inside herdr, Ghostty has to **intercept** the key and **send the bytes
herdr expects** instead. That is what `keybind = <key>=text:<bytes>` does:

```
keybind = cmd+a=text:\x02
         └─ what I press └─ bytes written to the focused pane, as if typed
```

`text:` accepts escape sequences: `\x02` is a byte in hex, `\r` is Enter, `\x1b` is ESC.

## Trick 1: Cmd -> a plain control byte

```
keybind = cmd+a=text:\x02
```

`\x02` is `ctrl+b`, herdr's **prefix**. Pressing `cmd+a` behaves exactly like `ctrl+b`.
`cmd+a` was picked over `cmd+b` for muscle memory, and because `cmd+←` already sends `\x01`
(`ctrl+a`), so there is no clash.

## Trick 2: Cmd -> Kitty "CSI u" sequence (modified keys)

Plain bytes cannot express `ctrl+shift+h`. The **Kitty keyboard protocol** can, and herdr
understands it. The format is:

```
ESC [ <codepoint> ; <modifiers + 1> u        ->   \x1b[104;6u
```

- **codepoint**: the Unicode value of the key (`h` = 104, `l` = 108, `p` = 112).
- **modifiers**: a bitmask where shift = 1, alt = 2, ctrl = 4. `ctrl+shift` = 1 + 4 = 5, and the
  protocol sends the mask **plus 1**, so `6`.

Used here:

| Ghostty key | Sends | herdr sees | Action |
| --- | --- | --- | --- |
| `cmd+alt+left` | `\x1b[104;6u` | `ctrl+shift+h` | previous tab |
| `cmd+alt+right` | `\x1b[108;6u` | `ctrl+shift+l` | next tab |
| `cmd+p` | `\x1b[112;6u` | `ctrl+shift+p` | switcher popup |

To add your own: pick the herdr key (for example `ctrl+shift+x`), look up the codepoint of the
letter, and send `\x1b[<codepoint>;6u`.

## Trick 3: Cmd -> a command

```
keybind = cmd+k=text:clear\r
```

Types `clear` and presses Enter in the focused pane. Ghostty's native `clear_screen` does not
work under herdr (the TUI repaints over it), so we ask the shell to do it. Caveat: it types into
whatever is focused, so do not press it while a program is waiting for input.

## Trick 4: `performable:` for Ctrl+hjkl

```
keybind = performable:ctrl+h=goto_split:left
```

`performable:` makes Ghostty consume the key **only if the action can actually happen**. At the
edge of the Ghostty splits (or inside herdr, where Ghostty sees one surface) the key falls
through to the program. One binding then works in three places: Ghostty splits, herdr panes
(via the `vim-herdr-navigation` plugin) and Neovim splits.

## Trick 5: Option as accent key, with escape hatches

```
macos-option-as-alt = false
keybind = alt+left=unbind
keybind = alt+right=unbind
keybind = alt+backspace=text:\x1b\x7f
```

With `macos-option-as-alt = false`, Option composes accents (useful for typing `ñ`, `é`, etc.) and
no longer sends Alt sequences. The cost: `alt+1..9` never reaches herdr, which is why herdr uses
`ctrl+shift` for agents. The three extra lines restore word navigation:

- `unbind` lets `alt+←/→` pass through to the shell.
- `\x1b\x7f` (ESC + DEL) is `backward-kill-word`.

## Trick 6: Shift+Enter inserts a newline

```
keybind = shift+enter=text:\x1b\r
```

Sends ESC + Enter, which TUIs and REPLs read as "newline without submitting".

## Debugging a binding

- Reload the config with `cmd+shift+,`, no restart needed.
- To see what bytes a key sends, run `cat -v` and press it (`^[` is ESC).
- If a Ghostty key does nothing in herdr, check that the herdr key exists in
  `config.toml` under `[keys]`, and that it is not already used by another binding.
