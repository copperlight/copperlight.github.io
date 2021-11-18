
## Introduction

Default [Z shell] keyboard shortcuts, tailored for MacOS and [Kitty].

I have been on a minimal configuration kick lately and don't want to customize things heavily.

Use `bindkey` to see current configuration settings. More details available in the `zshzle` man
page.

[Z shell]: http://zsh.sourceforge.net/
[Kitty]: https://sw.kovidgoyal.net/kitty/#

## Kitty Windows & Tabs

| Description | Shortcut |
|-------------|----------|
| Switch between enabled window layouts | `ctrl-shift-l` |
| New window | `cmd-enter` |
| Close window | `cmd-shift-d` |
| Next window | `ctrl-shift-]` |
| Previous window | `ctrl-shift-[` |
| Move window forward | `ctrl-shift-f` |
| Move window backward | `ctrl-shift-b` |
| New tab | `cmd-t` |
| Close tab | `cmd-w` |
| Next tab | `cmd-shift-]` |
| Previous tab | `cmd-shift-[` |
| Move tab forward | `ctrl-shift-.` |
| Move tab backward | `ctrl-shift-,` |

## Moving within a Line

| Description | Mapping | Shortcut |
|-------------|---------|----------|
| Move one character backwards | `backward-char` | `leftArrow` <br> `ctrl-b` |
| Move one character forwards | `forward-char` | `rightArrow` <br> `ctrl-f` |
| Move one word backwards | `backward-word` | `ctrl-[b` |
| Move one word forwards | `forward-word` | `ctrl-[f` |
| Move to the beginning of the line | `beginning-of-line` | `ctrl-a` |
| Move to the end of the line | `end-of-line` | `ctrl-e` |

## Editing a Line

| Description | Mapping | Shortcut |
|-------------|---------|----------|
| Delete the character before the cursor | `backward-delete-char` | `delete` <br> `ctrl-h` |
| Delete the character under the cursor | `delete-char-or-list` | `ctrl-d` |
| Delete the word before the cursor | `backward-kill-word` | `ctrl-w` <br> `ctrl-[,ctrl-h` |
| Delete the word after the cursor | `kill-word` | `ctrl-[d` |
| Delete the line after the cursor | `kill-line` | `ctrl-k` |
| Delete the whole line | `kill-whole-line` | `ctrl-u` |
| Transpose the two characters before the cursor | `transpose-chars` | `ctrl-t` |
| Transpose the two words before the cursor | `transpose-words` | `ctrl-[t` |
| Make a word lowercase | `down-case-word` | `ctrl-[l` |
| Make a word uppercase | `up-case-word` | `ctrl-[u` |
| Quote line | `quote-line` | `ctrl-['` |
| Push line onto buffer | `push-line` | `ctrl-[q` |
| Get line from buffer | `get-line` | `ctrl-[g` |
| Delete buffer | `kill-buffer` | `ctrl-x,ctrl-k` | 
| Undo the last change | `undo` | `ctrl-xu` <br> `ctrl-x,ctrl-u` |
| Execute line | `accept-line` | `enter` <br> `ctrl-j` <br> `ctrl-m` |

## Screen Management

| Description | Mapping | Shortcut |
|-------------|---------|----------|
| Clear screen, leaving current line intact | `clear-screen` | `ctrl-l` |
| Halt output to screen | ? | `ctrl-s` |
| Resume output to screen | ? | `ctrl-q` |

## Process Management

| Description | Shortcut |
|-------------|----------|
| Terminate/kill current foreground process | `ctrl-c` |
| Suspend/stop current foreground process | `ctrl-z` |
| Execute last command in history | `!!` |
| Execute last command in history that starts with abc | `!abc` |
| Print last command in history beginning with abc | `!abc:p` |

## History

| Description | Mapping | Shortcut |
|-------------|---------|----------|
| Previous history line | `up-line-or-history` | `upArrow` <br> `ctrl-p` |
| Next history line | `down-line-or-history` | `downArrow` <br> `ctrl-n` |
| Search history backwards | `history-incremental-search-backward` | `ctrl-r` |
| Search history forwards | `history-incremental-search-forward` | `ctrl-s` |
| Exit history search | `send-break` | `ctrl-g` |
