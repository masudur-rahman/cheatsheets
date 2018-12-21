# Basic VI Editor Commands

In VI editor two modes are available.

* Command mode
* Insert mode

## Text Entry commands :

* `a` - Append text following current cursor position
* `A` - Append to the end of current line
* `i` - Insert text before the current cursor position
* `I` - Insert text at the begining of the cursor position
* `o` - Insert a new line below the cursor line and start from there
* `O` - Insert a new line above the cursor line and start from there


## Cursor movement commands

* `j` or `<down-arrow>` or `<return>`- Move cursor down one line
* `k` or `<up-arrow>` - Move cursor up one line
* `h` or `<backspace>` or `<left-arrow>` - Move cursor one character to left
* `l` or `<space>` or `<right-arrow>` - Move cursor one character to right
* `0` or `^` - Move the cursor to start of the current line 
* `$` - Move the cursor to the end of the current line
* `w` - Move cursor to start of next word
* `b` - Move cursor to start of preceding word
* `:0<return>` or `:1<return>` or `1G` - Move cursor to start of the first line
* `:n<return>` or `nG` - Move cursor to start of the nth line
* `:$<return>` or `G` - Move cursor to the last line 



## Exit commands :
* `:q<return>` - Exit the editor
* `:q!<return>` - Exit the editor forcefully
* `:w<return>` - Save the contents in the file
* `:wq<return>` or `ZZ`- Save and exit, timestamp changes as re-saved even if file isn't changed
* `:x<return>` - Save and exit if file is changed otherwise just exit the editor



## Text Deletion commands :
* `x` - Delete single character
* `dw` - Delete current word forward from cursor postion
* `db` - Delete current word backward from cursor position
* `dd` - Delete the entire line
* `d$` or `D` - Delete to the end of the line from the cursor position
* `d^` - Delete to begining of line 


## Copy & Paste commands :

* `yy` - Copy the current line
* `Nyy` or `yNy` - Copy the next N lines including the current line
* `yw` - Copy from cursor to the end of current word
* `y$` - copy from cursor to the end of current line
* `p` - Paste below cursor 
* `P` - Paste above cursor
* `u` - Undo last change
* `U` - Restore line
* `J` - Join next line down to the end of the current line


## Replace commands :

* `r` - Replace one character at the cursor position
* `cw` - Replace current word to a new word


## Repeatation of last command :

* `.` - Repeat the last command


