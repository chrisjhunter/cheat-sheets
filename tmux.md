# tmux Cheat Sheet

> Default prefix is `Ctrl-b` (written as `C-b` below). Many configs remap to `C-a`.

## Sessions
```bash
tmux                       # start new session
tmux new -s work           # start named session
tmux ls                    # list sessions
tmux attach -t work        # attach to session
tmux attach                # attach to last session
tmux kill-session -t work  # kill a session
tmux kill-server           # kill all sessions
```
Inside tmux:
```
C-b d          detach from session
C-b s          list/switch sessions
C-b $          rename session
```

## Windows (tabs)
```
C-b c          create new window
C-b ,          rename window
C-b n / p      next / previous window
C-b 0-9        jump to window number
C-b w          list windows
C-b &          kill window
```

## Panes (splits)
```
C-b %          split vertically (side by side)
C-b "          split horizontally (stacked)
C-b arrow      move between panes
C-b o          cycle to next pane
C-b z          zoom/unzoom current pane (fullscreen toggle)
C-b x          kill current pane
C-b {  / C-b } swap pane left/right
C-b space      cycle pane layouts
```

## Resizing panes
```
C-b : resize-pane -D 10  # resize down by 10 cells
C-b : resize-pane -U 10  # resize up
C-b : resize-pane -L 10  # resize left
C-b : resize-pane -R 10  # resize right
```

## Copy mode / scrolling
```
C-b [          enter copy mode (scroll with arrows/PgUp)
q              exit copy mode
C-b ]          paste
```
In copy mode (vi-style, if `set -g mode-keys vi`): `v` to select, `y` to yank.

## Useful CLI one-liners
```bash
tmux new -d -s bg 'htop'               # detached session running a command
tmux send-keys -t work 'ls -la' Enter  # send command to a session
tmux capture-pane -t work -p           # dump pane content to stdout
tmux list-panes -a                     # list all panes across sessions
tmux source-file ~/.tmux.conf          # reload config
```

## Sample ~/.tmux.conf
```
set -g mouse on
set -g base-index 1
setw -g pane-base-index 1
set -g mode-keys vi
bind r source-file ~/.tmux.conf \; display "Reloaded!"
```

## Alternative: screen

`screen` predates tmux and is still preinstalled on more systems out of
the box — handy on a box you don't control or can't install packages on.
No panes/splits story as good as tmux, but the same core job (detachable
persistent sessions).

```bash
screen -S mysession     # start a new named session
screen -ls              # list sessions
screen -r mysession     # reattach
screen -r               # reattach to the only detached session
screen -d -r mysession  # detach it elsewhere first, then reattach here
screen -x mysession     # attach without detaching other clients (shared view)
```
Inside screen (prefix is `Ctrl-a` by default):
```
C-a d          detach
C-a c          new window
C-a n / p      next / previous window
C-a "          list windows
C-a A          rename window
C-a S          split horizontally
C-a |          split vertically (newer screen versions)
C-a Tab        switch between split regions
C-a k          kill current window/region
```
