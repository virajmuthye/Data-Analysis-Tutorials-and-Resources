# tmux Tutorial

**Author:** Viraj Rajendra Muthye  
**Email:** viraj.muthye@gmail.com

---

## What is tmux?

tmux is a **terminal multiplexer** — it lets you run multiple terminal sessions inside a single window, keep sessions alive after you disconnect (crucial for long-running jobs on HPC/servers), and organize work into named sessions and windows.

---

## 1. Starting a Named Session

Always name your sessions. It makes reattaching intuitive.

```bash
tmux new-session -s mysession
# or shorthand:
tmux new -s mysession
```

If tmux is already running and you want to create a new session from inside it:

```
Ctrl+b  :new-session -s mysession
```

---

## 2. Creating Windows Within a Session

Once inside a session, each **window** is like a tab in a browser.

```
Ctrl+b  c          # Create a new window
```

You'll see a window list appear at the bottom status bar, numbered `0`, `1`, `2`, etc.

---

## 3. Naming Windows

```
Ctrl+b  ,          # Rename the current window
```

A prompt appears at the bottom — type the name and hit Enter. Good names for a bioinformatics workflow might be: `pipeline`, `monitor`, `logs`, `db`.

---

## 4. Switching Between Windows

```
Ctrl+b  n          # Next window
Ctrl+b  p          # Previous window
Ctrl+b  0–9        # Jump directly to window by number (e.g., Ctrl+b 2)
Ctrl+b  l          # Last (most recently used) window
Ctrl+b  w          # Interactive window list — arrow keys + Enter to select
```

---

## 5. Detaching and Reattaching

This is tmux's killer feature — your session keeps running even when you close the terminal or SSH disconnects.

```bash
# Inside tmux, detach:
Ctrl+b  d

# From the shell, reattach:
tmux attach -t mysession
# or shorthand:
tmux a -t mysession

# If there's only one session:
tmux a
```

---

## Full Command Reference

### Session Management

| Command | Action |
|---|---|
| `tmux new -s name` | New named session |
| `tmux ls` | List all sessions |
| `tmux a -t name` | Attach to session |
| `tmux kill-session -t name` | Kill a session |
| `tmux rename-session -t old new` | Rename a session |
| `Ctrl+b  $` | Rename current session (from inside) |
| `Ctrl+b  d` | Detach from session |
| `Ctrl+b  s` | Interactive session list |

---

### Window Management

| Shortcut | Action |
|---|---|
| `Ctrl+b  c` | Create new window |
| `Ctrl+b  ,` | Rename current window |
| `Ctrl+b  &` | Kill current window (confirms first) |
| `Ctrl+b  n` | Next window |
| `Ctrl+b  p` | Previous window |
| `Ctrl+b  0–9` | Jump to window by number |
| `Ctrl+b  w` | Visual window picker |
| `Ctrl+b  l` | Last active window |
| `Ctrl+b  f` | Find window by name |

---

### Pane Management (splitting windows)

A window can be split into multiple **panes** — great for watching logs while running commands.

| Shortcut | Action |
|---|---|
| `Ctrl+b  %` | Split vertically (side by side) |
| `Ctrl+b  "` | Split horizontally (top and bottom) |
| `Ctrl+b  ←↑↓→` | Move between panes |
| `Ctrl+b  o` | Cycle to next pane |
| `Ctrl+b  z` | Zoom current pane (toggle fullscreen) |
| `Ctrl+b  x` | Kill current pane |
| `Ctrl+b  q` | Show pane numbers (then press the number to jump) |
| `Ctrl+b  {` | Swap pane left |
| `Ctrl+b  }` | Swap pane right |
| `Ctrl+b  Space` | Cycle through pane layouts |

---

### Scrolling and Copy Mode

By default you can't scroll up. Copy mode fixes that.

| Shortcut | Action |
|---|---|
| `Ctrl+b  [` | Enter copy/scroll mode |
| Arrow keys or `PgUp/PgDn` | Scroll |
| `/` | Search forward (in copy mode) |
| `?` | Search backward (in copy mode) |
| `q` | Exit copy mode |
| `Ctrl+b  ]` | Paste from tmux buffer |

---

### Miscellaneous

| Shortcut | Action |
|---|---|
| `Ctrl+b  ?` | Show all keybindings (your cheat sheet) |
| `Ctrl+b  t` | Show a clock |
| `Ctrl+b  :` | Enter tmux command prompt |
| `Ctrl+b  r` | Reload config (if set up) |

---

## Practical Workflow Example

```bash
# Start a session for your run monitor
tmux new -s runmonitor

# Rename the first window
# Ctrl+b ,  →  "gateway"

# Open a new window for logs
# Ctrl+b c
# Ctrl+b ,  →  "logs"

# Open another for DB / psql
# Ctrl+b c
# Ctrl+b ,  →  "db"

# Split the logs window to watch celery + uvicorn side by side
# Ctrl+b %

# Detach when done
# Ctrl+b d

# Come back later
tmux a -t runmonitor
```

---

## Pro Tips

- **Never kill the terminal — always detach.** `Ctrl+b d` keeps everything running.
- `tmux ls` is your first command after SSH-ing into a server — always check what's already running.
- You can have multiple sessions per server (e.g., one per project).
- Add `set -g mouse on` to `~/.tmux.conf` to enable mouse scrolling and pane clicking.
- `Ctrl+b z` (zoom) is underrated — instantly fullscreens a pane, toggle it back the same way.

---

## Resources

- [tmux GitHub Wiki](https://github.com/tmux/tmux/wiki)
- [tmux Cheat Sheet](https://tmuxcheatsheet.com/)
- ["A tmux Crash Course" (Thoughtbot)](https://thoughtbot.com/blog/a-tmux-crash-course)

---

For questions, suggestions, or contributions, please contact:
**Viraj Rajendra Muthye** - viraj.muthye@gmail.com

---

*Happy multiplexing!*
