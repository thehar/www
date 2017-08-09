+++
description = "tmux love"
categories = ["tools"]
draft = true
author = ""
+++

# tmux love and you

new session: tmux
start new with fancy name: tmux new -s myname

attach: tmux a #
attach to fancy named session: tmux a -t myname

list sessions: tmux ls

kill session: tmux kill-session -t myname
Kill all the tmux sessions: tmux ls | grep : | cut -d. -f1 | awk '{print substr($1, 0, length($1)-1)}' | xargs kill
