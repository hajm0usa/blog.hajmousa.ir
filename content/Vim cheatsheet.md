+++
title="Complete Vim Cheatsheet"
date=2025-01-12
+++

## Modes

Vim has several modes that change how keys behave:

- **Normal Mode** - Default mode for navigation and commands (press `Esc` to return here)
- **Insert Mode** - For typing text (press `i` to enter)
- **Visual Mode** - For selecting text (press `v` to enter)
- **Command Mode** - For executing commands (press `:` to enter)
- **Replace Mode** - For overwriting text (press `R` to enter)

## Starting and Exiting Vim

- `vim filename` - Open or create a file
- `vim +N filename` - Open file at line N
- `vim +/pattern filename` - Open file at first occurrence of pattern
- `:q` - Quit (fails if unsaved changes)
- `:q!` - Quit without saving
- `:wq` or `:x` or `ZZ` - Save and quit
- `:w` - Save file
- `:w filename` - Save as filename
- `:w!` - Force save (overwrite read-only)
- `ZQ` - Quit without saving

## Movement (Normal Mode)

### Basic Movement

- `h` - Move left
- `j` - Move down
- `k` - Move up
- `l` - Move right
- `w` - Move forward to start of next word
- `W` - Move forward to start of next WORD (space-separated)
- `e` - Move forward to end of word
- `E` - Move forward to end of WORD
- `b` - Move backward to start of word
- `B` - Move backward to start of WORD
- `ge` - Move backward to end of previous word

### Line Movement

- `0` - Move to beginning of line
- `^` - Move to first non-blank character of line
- `$` - Move to end of line
- `g_` - Move to last non-blank character of line
- `+` or `Enter` - Move to first character of next line
- `-` - Move to first character of previous line

### Screen Movement

- `H` - Move to top of screen (High)
- `M` - Move to middle of screen (Middle)
- `L` - Move to bottom of screen (Low)
- `Ctrl+f` - Move forward one full screen (Page Down)
- `Ctrl+b` - Move backward one full screen (Page Up)
- `Ctrl+d` - Move forward half screen (Down)
- `Ctrl+u` - Move backward half screen (Up)
- `Ctrl+e` - Scroll down one line
- `Ctrl+y` - Scroll up one line
- `zt` - Scroll current line to top of screen
- `zz` - Scroll current line to center of screen
- `zb` - Scroll current line to bottom of screen

### Document Movement

- `gg` - Go to first line of document
- `G` - Go to last line of document
- `NG` or `:N` - Go to line N
- `%` - Jump to matching bracket/parenthesis

### Search Movement

- `f{char}` - Move forward to next occurrence of {char} on line
- `F{char}` - Move backward to previous occurrence of {char} on line
- `t{char}` - Move forward to just before next occurrence of {char}
- `T{char}` - Move backward to just after previous occurrence of {char}
- `;` - Repeat last f, F, t, or T command forward
- `,` - Repeat last f, F, t, or T command backward

## Inserting Text

- `i` - Insert before cursor
- `I` - Insert at beginning of line
- `a` - Insert after cursor (append)
- `A` - Insert at end of line
- `o` - Open new line below and insert
- `O` - Open new line above and insert
- `ea` - Insert at end of word
- `gi` - Insert at last insert position
- `Esc` or `Ctrl+[` - Exit insert mode

## Editing

### Deleting

- `x` - Delete character under cursor
- `X` - Delete character before cursor
- `dw` - Delete from cursor to start of next word
- `de` - Delete from cursor to end of word
- `d$` or `D` - Delete from cursor to end of line
- `d0` - Delete from cursor to beginning of line
- `dd` - Delete entire line
- `Ndd` - Delete N lines
- `dG` - Delete from current line to end of file
- `dgg` - Delete from current line to beginning of file
- `:Nd` - Delete line N

### Changing (Delete and Enter Insert Mode)

- `cw` - Change from cursor to start of next word
- `ce` - Change from cursor to end of word
- `c$` or `C` - Change from cursor to end of line
- `cc` or `S` - Change entire line
- `s` - Delete character and enter insert mode
- `r{char}` - Replace single character with {char}
- `R` - Enter replace mode

### Copy and Paste

- `yy` or `Y` - Yank (copy) entire line
- `Nyy` - Yank N lines
- `yw` - Yank word
- `y$` - Yank from cursor to end of line
- `p` - Paste after cursor
- `P` - Paste before cursor
- `gp` - Paste after cursor and move cursor after pasted text
- `gP` - Paste before cursor and move cursor after pasted text
- `:reg` - View contents of all registers
- `"ay` - Yank into register a
- `"ap` - Paste from register a

### Undo and Redo

- `u` - Undo last change
- `U` - Undo all changes on current line
- `Ctrl+r` - Redo
- `.` - Repeat last command

### Text Manipulation

- `J` - Join current line with next line
- `gJ` - Join lines without adding space
- `~` - Switch case of character under cursor
- `g~w` - Switch case of word
- `guw` - Make word lowercase
- `gUw` - Make word uppercase
- `>>` - Indent line
- `<<` - Unindent line
- `==` - Auto-indent line
- `gg=G` - Auto-indent entire file

## Visual Mode

- `v` - Enter character-wise visual mode
- `V` - Enter line-wise visual mode
- `Ctrl+v` - Enter block-wise visual mode
- `o` - Move to other end of selection
- `O` - Move to other corner of block
- `aw` - Select a word
- `ab` - Select a block with ()
- `aB` - Select a block with {}
- `at` - Select a block with <> tags
- `ib` - Select inner block with ()
- `iB` - Select inner block with {}
- `it` - Select inner block with <> tags

### Visual Mode Commands

After selecting text in visual mode:

- `d` - Delete selection
- `c` - Change selection
- `y` - Yank selection
- `>` - Indent selection
- `<` - Unindent selection
- `=` - Auto-indent selection
- `~` - Switch case
- `u` - Make lowercase
- `U` - Make uppercase

## Search and Replace

### Searching

- `/pattern` - Search forward for pattern
- `?pattern` - Search backward for pattern
- `n` - Repeat search in same direction
- `N` - Repeat search in opposite direction
- `*` - Search forward for word under cursor
- `#` - Search backward for word under cursor
- `g*` - Search forward for partial word under cursor
- `g#` - Search backward for partial word under cursor
- `:noh` - Clear search highlighting

### Search Options

- `/pattern\c` - Case-insensitive search
- `/pattern\C` - Case-sensitive search
- `/\<pattern\>` - Search for whole word only

### Replace

- `:s/old/new` - Replace first occurrence in current line
- `:s/old/new/g` - Replace all occurrences in current line
- `:s/old/new/gc` - Replace all with confirmation in current line
- `:%s/old/new/g` - Replace all occurrences in entire file
- `:%s/old/new/gc` - Replace all with confirmation in entire file
- `:5,12s/old/new/g` - Replace all occurrences between lines 5-12
- `:'<,'>s/old/new/g` - Replace all in visual selection

### Replace Flags

- `g` - Replace all occurrences in line
- `c` - Confirm each substitution
- `i` - Case-insensitive
- `I` - Case-sensitive

## Marks and Jumps

- `m{a-z}` - Set mark {a-z} in current file
- `m{A-Z}` - Set mark {A-Z} globally across files
- `` `{a-z} `` - Jump to mark {a-z}
- `'{a-z}` - Jump to start of line with mark {a-z}
- ` `` ` - Jump to position before last jump
- `''` - Jump to start of line before last jump
- `Ctrl+o` - Jump to previous location in jump list
- `Ctrl+i` - Jump to next location in jump list
- `:marks` - List all marks
- `:delmarks a` - Delete mark a
- `:delmarks!` - Delete all lowercase marks

## Macros

- `q{a-z}` - Start recording macro into register {a-z}
- `q` - Stop recording macro
- `@{a-z}` - Execute macro from register {a-z}
- `@@` - Execute last executed macro
- `N@{a-z}` - Execute macro N times

## Working with Multiple Files

### Buffers

- `:e filename` - Edit a file
- `:bn` - Go to next buffer
- `:bp` - Go to previous buffer
- `:bd` - Delete buffer (close file)
- `:b#` - Toggle between current and last buffer
- `:b N` - Go to buffer N
- `:ls` or `:buffers` - List all buffers
- `:badd filename` - Add file to buffer list without opening
- `:ball` - Open all buffers in windows

### Windows (Splits)

- `:split` or `:sp` - Split window horizontally
- `:vsplit` or `:vs` - Split window vertically
- `:sp filename` - Open filename in horizontal split
- `:vsp filename` - Open filename in vertical split
- `Ctrl+w s` - Split window horizontally
- `Ctrl+w v` - Split window vertically
- `Ctrl+w h` - Move to left window
- `Ctrl+w j` - Move to window below
- `Ctrl+w k` - Move to window above
- `Ctrl+w l` - Move to right window
- `Ctrl+w w` - Cycle through windows
- `Ctrl+w q` - Quit window
- `Ctrl+w =` - Make all windows equal size
- `Ctrl+w _` - Maximize current window height
- `Ctrl+w |` - Maximize current window width
- `Ctrl+w +` - Increase window height
- `Ctrl+w -` - Decrease window height
- `Ctrl+w >` - Increase window width
- `Ctrl+w <` - Decrease window width

### Tabs

- `:tabnew` or `:tabe` - Open new tab
- `:tabnew filename` - Open filename in new tab
- `:tabn` or `gt` - Go to next tab
- `:tabp` or `gT` - Go to previous tab
- `:tabfirst` - Go to first tab
- `:tablast` - Go to last tab
- `:tabclose` or `:tabc` - Close current tab
- `:tabonly` - Close all other tabs
- `:tabs` - List all tabs
- `Ngt` - Go to tab N

## Folding

- `zf` - Create fold
- `zd` - Delete fold under cursor
- `zD` - Delete all folds under cursor recursively
- `zo` - Open fold
- `zO` - Open all folds under cursor recursively
- `zc` - Close fold
- `zC` - Close all folds under cursor recursively
- `za` - Toggle fold
- `zA` - Toggle all folds under cursor recursively
- `zr` - Reduce folding by opening one more level
- `zR` - Open all folds
- `zm` - More folding by closing one more level
- `zM` - Close all folds
- `zj` - Move to next fold
- `zk` - Move to previous fold

## Text Objects

Text objects are used with operators (d, c, y, etc.) in the format: `[operator][count][text-object]`

### Inner Text Objects (i)

- `iw` - Inner word
- `iW` - Inner WORD
- `is` - Inner sentence
- `ip` - Inner paragraph
- `i"` - Inner quoted string (double quotes)
- `i'` - Inner quoted string (single quotes)
- `i`` - Inner quoted string (backticks)
- `i(` or `i)` or `ib` - Inner parentheses block
- `i{` or `i}` or `iB` - Inner braces block
- `i[` or `i]` - Inner brackets block
- `i<` or `i>` - Inner angle brackets block
- `it` - Inner tag block (HTML/XML)

### Around Text Objects (a)

Includes the surrounding delimiters/whitespace:

- `aw` - Around word
- `aW` - Around WORD
- `as` - Around sentence
- `ap` - Around paragraph
- `a"` - Around quoted string (double quotes)
- `a'` - Around quoted string (single quotes)
- `a`` - Around quoted string (backticks)
- `a(` or `a)` or `ab` - Around parentheses block
- `a{` or `a}` or `aB` - Around braces block
- `a[` or `a]` - Around brackets block
- `a<` or `a>` - Around angle brackets block
- `at` - Around tag block (HTML/XML)

### Examples

- `diw` - Delete inner word
- `ci"` - Change inner quoted string
- `ya{` - Yank around braces block
- `dap` - Delete around paragraph
- `vit` - Visually select inner tag

## Command Mode

### File Commands

- `:w` - Save file
- `:w filename` - Save as filename
- `:wa` - Save all files
- `:q` - Quit
- `:qa` - Quit all
- `:wqa` - Save and quit all
- `:q!` - Force quit without saving
- `:r filename` - Read file and insert below cursor
- `:r !command` - Read output of command and insert below cursor
- `:saveas filename` - Save as filename and start editing new file

### Line Numbers and Ranges

- `:N` - Jump to line N
- `:$` - Jump to last line
- `:.` - Current line
- `:.,+N` - Current line to N lines below
- `:%` - All lines (entire file)
- `:'<,'>` - Visual selection
- `:5,10` - Lines 5 to 10
- `:.,/pattern/` - Current line to line matching pattern

### Editing Commands

- `:d` - Delete line(s)
- `:y` - Yank line(s)
- `:t` or `:co` - Copy line(s) to position
- `:m` - Move line(s) to position
- `:g/pattern/d` - Delete all lines matching pattern
- `:g!/pattern/d` or `:v/pattern/d` - Delete all lines NOT matching pattern
- `:sort` - Sort lines
- `:sort!` - Sort lines in reverse
- `:sort u` - Sort lines and remove duplicates

### External Commands

- `:!command` - Execute external command
- `:r !command` - Read command output into file
- `:w !command` - Write file content to command
- `:%!command` - Filter entire file through command
- `:5,10!command` - Filter lines 5-10 through command

### Settings

- `:set option` - Enable option
- `:set nooption` - Disable option
- `:set option?` - Check option value
- `:set option=value` - Set option value
- `:set` - Show all non-default options
- `:set all` - Show all options

### Common Options

- `:set number` or `:set nu` - Show line numbers
- `:set nonumber` or `:set nonu` - Hide line numbers
- `:set relativenumber` or `:set rnu` - Show relative line numbers
- `:set norelativenumber` or `:set nornu` - Hide relative line numbers
- `:set hlsearch` - Highlight search results
- `:set nohlsearch` - Don't highlight search results
- `:set incsearch` - Incremental search
- `:set ignorecase` or `:set ic` - Case-insensitive search
- `:set smartcase` or `:set scs` - Case-sensitive if uppercase present
- `:set wrap` - Wrap lines
- `:set nowrap` - Don't wrap lines
- `:set autoindent` - Auto-indent new lines
- `:set expandtab` - Use spaces instead of tabs
- `:set tabstop=N` - Set tab width to N spaces
- `:set shiftwidth=N` - Set indent width to N spaces
- `:set syntax=language` - Set syntax highlighting language
- `:syntax on` - Enable syntax highlighting
- `:syntax off` - Disable syntax highlighting

## Help and Documentation

- `:help` or `:h` - Open help
- `:help keyword` - Help on keyword
- `:help :command` - Help on command
- `:help option` - Help on option
- `Ctrl+]` - Jump to tag under cursor in help
- `Ctrl+t` - Jump back in help
- `:helpgrep pattern` - Search all help files for pattern
- `:cn` - Next help search result
- `:cp` - Previous help search result

## Miscellaneous

### Completion (Insert Mode)

- `Ctrl+n` - Next completion suggestion
- `Ctrl+p` - Previous completion suggestion
- `Ctrl+x Ctrl+f` - File name completion
- `Ctrl+x Ctrl+l` - Line completion
- `Ctrl+x Ctrl+n` - Keyword completion (current file)
- `Ctrl+x Ctrl+o` - Omni completion (context-aware)

### Other Useful Commands

- `ga` - Show ASCII value of character under cursor
- `g8` - Show UTF-8 encoding of character under cursor
- `gf` - Open file whose name is under cursor
- `Ctrl+g` - Show file information (name, lines, position)
- `g Ctrl+g` - Show detailed file statistics
- `:!` - Repeat last external command
- `K` - Look up keyword under cursor in man pages
- `:earlier 5m` - Go to file state 5 minutes ago
- `:later 5m` - Go to file state 5 minutes later
- `:retab` - Replace tabs with spaces (or vice versa)
- `:TOhtml` - Convert file to HTML

## Configuration

Vim can be configured via the `.vimrc` file (or `_vimrc` on Windows) in your home directory.

### Sample .vimrc Settings

```vim
" Enable syntax highlighting
syntax on

" Show line numbers
set number

" Enable mouse support
set mouse=a

" Set tab width
set tabstop=4
set shiftwidth=4
set expandtab

" Enable auto-indent
set autoindent
set smartindent

" Highlight search results
set hlsearch
set incsearch

" Case-insensitive search unless uppercase present
set ignorecase
set smartcase

" Show matching brackets
set showmatch

" Enable line wrapping
set wrap

" Show command in bottom bar
set showcmd

" Visual autocomplete for command menu
set wildmenu

" Disable backup files
set nobackup
set noswapfile
```

## Tips and Tricks

1. **Combine operators with counts and motions**: `d3w` deletes 3 words, `c2j` changes 2 lines down
2. **Use `.` to repeat**: The dot command repeats your last change
3. **Use `*` for quick searching**: Place cursor on word and press `*` to search
4. **Visual block mode is powerful**: Use `Ctrl+v` for column editing
5. **Learn text objects**: Commands like `ci"` (change inside quotes) are very efficient
6. **Use marks for quick navigation**: Set marks with `ma` and jump with `` `a ``
7. **Record complex changes as macros**: Use `qa` to start recording, `q` to stop, `@a` to replay
8. **Use `:g` for global commands**: `:g/pattern/d` deletes all lines matching pattern
9. **Split windows for editing multiple files**: `Ctrl+w s` and `Ctrl+w v`
10. **Use registers for multiple clipboards**: Yank to `"ay` and paste from `"ap`
