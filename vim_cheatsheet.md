# vim_cheatsheet
# MODES
---
i → Enter INSERT mode (before cursor)
I → Enter INSERT mode (beginning of line)
a → Enter INSERT mode (after cursor)
A → Enter INSERT mode (end of line)
o → Open new line below & enter INSERT
O → Open new line above & enter INSERT
v → Enter VISUAL mode (character select)
V → Enter VISUAL mode (line select)
Ctrl+v → Enter VISUAL BLOCK mode (column select)
R → Enter REPLACE mode (overwrite text)
Esc → Return to NORMAL mode
: → Enter COMMAND mode
---
## NAVIGATION — BASIC
h → Move left
j → Move down
k → Move up
l → Move right
w → Jump to start of next word
W → Jump to start of next WORD (whitespace-delimited)
b → Jump to start of previous word
B → Jump to start of previous WORD
e → Jump to end of current/next word
0 → Jump to beginning of line
^ → Jump to first non-blank character of line
$ → Jump to end of line
---
## NAVIGATION — LINES & FILE
gg → Go to first line of file
G → Go to last line of file
:n → Go to line n (e.g. :42)
nG → Go to line n (e.g. 42G)
H → Move cursor to top of screen
M → Move cursor to middle of screen
L → Move cursor to bottom of screen
Ctrl+u → Scroll up half a page
Ctrl+d → Scroll down half a page
Ctrl+b → Scroll up full page
Ctrl+f → Scroll down full page
zz → Center current line on screen
zt → Move current line to top of screen
zb → Move current line to bottom of screen
---
## NAVIGATION — ADVANCED
% → Jump to matching bracket ( ) [ ] { }
f{char} → Jump forward to {char} on current line
F{char} → Jump backward to {char} on current line
t{char} → Jump forward to just before {char}
T{char} → Jump backward to just after {char}
; → Repeat last f/F/t/T forward
, → Repeat last f/F/t/T backward
* → Jump to next occurrence of word under cursor
# → Jump to previous occurrence of word under cursor
{ → Jump to previous empty line (paragraph up)
} → Jump to next empty line (paragraph down)
Ctrl+o → Jump to previous cursor position
Ctrl+i → Jump to next cursor position
---
## EDITING — DELETE
x → Delete character under cursor
X → Delete character before cursor
dw → Delete from cursor to start of next word
db → Delete from cursor to start of previous word
dd → Delete entire line
D → Delete from cursor to end of line
d$ → Delete from cursor to end of line (same as D)
d0 → Delete from cursor to beginning of line
d^ → Delete from cursor to first non-blank character
dG → Delete from current line to end of file
dgg → Delete from current line to beginning of file
5dd → Delete 5 lines
---
## EDITING — CHANGE
cw → Change word (delete word & enter INSERT)
cc → Change entire line
C → Change from cursor to end of line
ci" → Change inside double quotes
ci( → Change inside parentheses
ci{ → Change inside curly braces
ci[ → Change inside square brackets
ciw → Change inner word
caw → Change a word (including surrounding space)
ct{char} → Change up to {char}
s → Substitute character (delete & enter INSERT)
S → Substitute entire line
r{char} → Replace single character with {char}
~ → Toggle case of character under cursor
---
## EDITING — COPY & PASTE
yy → Yank (copy) entire line
yw → Yank word
y$ → Yank from cursor to end of line
yG → Yank from current line to end of file
p → Paste after cursor
P → Paste before cursor
"*yy → Yank line to system clipboard
"*p → Paste from system clipboard
:%y+ → Yank entire file to system clipboard
"+p → Paste from system clipboard (alternate)
---
## EDITING — INDENT & FORMAT
`>>` → Indent line right
`<<` → Indent line left
`>}` → Indent from cursor to next paragraph
`<}` → Unindent from cursor to next paragraph
== → Auto-indent current line
gg=G → Auto-indent entire file
:set tabstop=4 → Set tab width to 4
:set expandtab → Use spaces instead of tabs
:set shiftwidth=4 → Set indent width to 4
---
## VISUAL MODE OPERATIONS
v → Start character-wise selection
V → Start line-wise selection
Ctrl+v → Start block (column) selection
y → Yank selected text
d → Delete selected text
`>` → Indent selected text
`<` → Unindent selected text
= → Auto-indent selection
U → Uppercase selection
u → Lowercase selection
gv → Reselect last visual selection
ggVG → Select entire file
---
## UNDO & REDO
u → Undo last change
Ctrl+r → Redo last undone change
U → Undo all changes on current line
. → Repeat last action
5. → Repeat last action 5 times
---
## SEARCH & REPLACE
/pattern → Search forward for pattern
?pattern → Search backward for pattern
n → Jump to next match
N → Jump to previous match
:noh → Clear search highlighting
:s/old/new/ → Replace first on current line
:s/old/new/g → Replace all on current line
:%s/old/new/g → Replace all in entire file
:%s/old/new/gc → Replace all with confirmation
:%s/old/new/gi → Replace all case-insensitive
---
## FILES & BUFFERS
:w → Save current file
:w filename → Save as filename
:q → Quit (fails if unsaved changes)
:q! → Force quit (discard changes)
:wq → Save and quit
:x → Save and quit (same as :wq)
ZZ → Save and quit (normal mode shortcut)
ZQ → Quit without saving (normal mode shortcut)
:e filename → Open file in current buffer
:e! → Reload current file (discard changes)
:bn → Next buffer
:bp → Previous buffer
:bd → Close current buffer
:ls → List all open buffers
---
## SPLITS & TABS
:split filename → Horizontal split
:vsplit filename → Vertical split
Ctrl+w s → Horizontal split (current file)
Ctrl+w v → Vertical split (current file)
Ctrl+w h → Move to left split
Ctrl+w j → Move to split below
Ctrl+w k → Move to split above
Ctrl+w l → Move to right split
Ctrl+w w → Cycle through splits
Ctrl+w = → Equal size all splits
Ctrl+w q → Close current split
:tabnew filename → Open file in new tab
gt → Next tab
gT → Previous tab
:tabclose → Close current tab
---
## MACROS
q{key} → Start recording macro to register {key}
q → Stop recording
@{key} → Play macro from register {key}
@@ → Replay last played macro
5@{key} → Play macro 5 times
:reg → View all registers (including macros)
---
## MARKS & JUMPS
m{key} → Set mark at current position
'{key} → Jump to line of mark
`{key} → Jump to exact position of mark
:marks → List all marks
'' → Jump to last position before jump
'. → Jump to last edited line
---
## LINE OPERATIONS
J → Join current line with next line
:sort → Sort all lines alphabetically
:sort! → Sort in reverse
:sort u → Sort & remove duplicates
:%d → Delete all lines in file
:g/pattern/d → Delete all lines matching pattern
:v/pattern/d → Delete all lines NOT matching pattern
---
## USEFUL COMMANDS
:set number → Show line numbers
:set relativenumber → Show relative line numbers
:set nonumber → Hide line numbers
:set paste → Paste mode (disables auto-indent)
:set nopaste → Exit paste mode
:set ignorecase → Case-insensitive search
:set smartcase → Case-sensitive if uppercase present
:set hlsearch → Highlight all search matches
:set incsearch → Incremental search (highlight as you type)
:syntax on → Enable syntax highlighting
:set wrap → Enable line wrapping
:set nowrap → Disable line wrapping
---
## COMMON PATTERNS
Ctrl+u → Delete line before cursor in INSERT mode
Ctrl+w → Delete previous word in INSERT mode
Ctrl+a → Increment number under cursor
Ctrl+x → Decrement number under cursor
:!command → Run shell command from vim
:r !command → Insert output of shell command
:r filename → Insert contents of file
gf → Open file under cursor
:set spell → Enable spell checking
]s → Jump to next misspelled word
z= → Show spelling suggestions
