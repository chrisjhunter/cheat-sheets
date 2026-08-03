# Vim Cheat Sheet

## Modes
```
Esc            return to normal mode
i / a          insert before / after cursor
I / A          insert at line start / end
o / O          open new line below / above, enter insert mode
v              visual (character) mode
V              visual line mode
Ctrl-v         visual block mode
:              command mode
```

## Saving & quitting
```
:w             save
:w file.txt    save as
:q             quit
:q!            quit without saving
:wq  or  :x    save and quit
:wqa           save and quit all open buffers/windows
ZZ             save and quit (normal mode shortcut)
```

## Movement
```
h j k l        left, down, up, right
w / b          next / previous word start
e              end of word
0 / ^          start of line / first non-blank char
$              end of line
gg / G         start / end of file
:42            go to line 42  (or  42G)
Ctrl-d / Ctrl-u  half page down / up
Ctrl-f / Ctrl-b  full page down / up
%              jump to matching bracket/paren
```

## Editing
```
x              delete char under cursor
dd             delete (cut) line
dw             delete word
D              delete to end of line
yy             yank (copy) line
yw             yank word
p / P          paste after / before cursor
u              undo
Ctrl-r         redo
.              repeat last change
r<char>        replace single char
cw             change word (delete + insert)
cc             change whole line
J              join line below with current
~              toggle case of char under cursor
```

## Visual mode operations
```
v then move, then d      delete selection
v then move, then y      yank selection
V then move, then >      indent selected lines
gv                        reselect last visual selection
```

## Search & replace
```
/pattern           search forward
?pattern           search backward
n / N               next / previous match
:%s/foo/bar/g       replace all "foo" with "bar" in file
:%s/foo/bar/gc      same, with confirmation per match
:s/foo/bar/         replace first match on current line only
:5,10s/foo/bar/g    replace within line range 5-10
*                   search word under cursor, forward
```

## Multiple files / windows
```
:e file.txt        open another file
:bn / :bp          next / previous buffer
:ls                list open buffers
:sp file.txt        horizontal split
:vsp file.txt         vertical split
Ctrl-w w              cycle between windows
Ctrl-w q                close current window
```

## Marks & registers
```
ma                 set mark "a" at cursor
`a                  jump to mark "a"
"ayy                yank line into register a
"ap                   paste from register a
:reg                    list all registers
```

## Useful one-liners
```bash
vim +42 file.txt                     # open file, cursor at line 42
vim +/pattern file.txt                 # open file, cursor at first match
vim -d file1 file2                       # diff mode between two files
vimdiff file1 file2                        # same as above
vim -R file.txt                              # open read-only
```

## Config essentials (~/.vimrc)
```vim
set number                " line numbers
set relativenumber        " relative line numbers
set expandtab             " spaces instead of tabs
set shiftwidth=2
set tabstop=2
set incsearch             " live search highlighting
set ignorecase smartcase  " smart case-insensitive search
syntax on
```
