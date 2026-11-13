# herdr-goto

A tree navigator popup for [herdr](https://herdr.dev): machine › workspace › pane,
with live agent status, fuzzy search, vim-style keys, a "last workspace" jump,
pane preview and git worktrees. A Rust/ratatui re-implementation of herdr's
built-in `goto` widget, extended with the things I kept wishing it had.

```
 / search agents and terminals                              ● live  80 terminals

 ▾ Local
   ▾ dotfiles ●                                                            main
     ├─ ○ fix ls on macOS · 1                                   claude   working
     ├─ · Sidebar · 1                                           terminal shell
     └─ ▫ ⎇ feature/x  ~/dotfiles-wt/feature-x                  worktree closed
 ▾ gx10                                                                       ●
   ▸ leon_bot  (6)                                              idle
```

## Features

- **Live**: subscribes to herdr's event stream; statuses, titles, panes and
  workspaces update while the popup is open.
- **Saved SSH machines**: every `herdr machine` shows up under its own node.
  The last snapshot is cached, so remote trees appear instantly (`◌ cached`)
  and refresh in the background.
- **Fuzzy search** (`/`) over `workspace › title`, ranked, with highlighted
  matches; agent kind and status match too.
- **Status filters** `a` `b` `w` `i` `d` (all / blocked / working / idle / done),
  remembered across launches.
- **Vim keys**: `j`/`k`, `gg`/`G`, `C-d`/`C-u`, `H`/`M`/`L`, `h`/`l` + `za`/`zM`/`zR`
  for folding, `?` for the full list.
- **Last workspace** (`'` / `Tab`): bounce between the most recently used
  workspace and the current pane, like vim's `Ctrl-^`.
- **Preview** (`p`): the selected pane's recent output beside the tree.
- **Worktrees**: workspaces that are linked worktrees show `⎇ branch`; worktrees
  herdr has not opened yet are listed under their repo and `Enter` opens them.

## Install

With [mise](https://mise.jdx.dev) (downloads the prebuilt binary from the
GitHub release for your platform):

```sh
mise use -g github:king0980692/herdr-goto@latest
```

From source:

```sh
cargo install --git https://github.com/king0980692/herdr-goto
```

## Bind it in herdr

Add to `~/.config/herdr/config.toml`. Keybinding commands run on whichever
machine's server the TUI is showing, so install the binary and this binding on
each machine you switch to.

```toml
[keys]
goto = ""   # free alt+s from the built-in navigator (optional)

[[keys.command]]
key = "alt+s"
type = "popup"
description = "Go to"
command = "herdr-goto"
width = "80%"
height = "80%"
```

## Keys

| Key | Action |
| --- | --- |
| `j` / `k`, `↑` / `↓` | move |
| `C-d` / `C-u`, `C-f` / `C-b` | half / full page |
| `gg` / `G`, `H` / `M` / `L` | first / last, top / middle / bottom of view |
| `[` / `]` | previous / next workspace |
| `'` `` ` `` `Tab` | last-used workspace ⇄ current pane |
| `h` / `l` | parent · fold / unfold · first child |
| `za` `zo` `zc` `zM` `zR` | fold toggle / open / close / all / none |
| `/` | search (insert mode); `Esc` back to normal keeping the filter |
| `a` `b` `w` `i` `d`, `x` | status filter, clear |
| `p` | toggle preview |
| `Enter` | focus pane / workspace, open worktree, fold machine |
| `?` | key help |
| `q` / `Esc` | close |

## How it talks to herdr

Everything goes through herdr's socket API (`$HERDR_SOCKET_PATH`):
`session.snapshot`, `events.subscribe`, `pane.focus`, `pane.read`,
`worktree.list`, `worktree.open`. Remote machines are reached with
`herdr --machine <name> …`. State lives in `~/.cache/<app>/goto/`
(remote snapshot cache, MRU log, filter and preview preferences).

Jumping to a pane on another machine writes `client/jump-request.json` in
the app's state dir (`$HERDR_STATE_DIR`, else `~/.local/state/<app>`); the
herdr client switches machine and focuses it.

### herdr-native and other Herdr apps

Inside a Herdr pane or popup, goto follows that app: its socket
(`$HERDR_SOCKET_PATH`), binary (`$HERDR_BIN_PATH`) and state dir. Outside
Herdr (e.g. a quick terminal) there is no such env, so set the app:

```sh
HERDR_GOTO_APP=herdr-native herdr-goto
```

This uses `~/.config/herdr-native/herdr.sock`, `~/.local/state/herdr-native`, `~/.cache/herdr-native`
and the `herdr-native` binary (XDG dirs respected).

## License

MIT
