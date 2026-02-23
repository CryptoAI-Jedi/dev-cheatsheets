# vim_cheatsheet

---

## MODES
i          → Enter INSERT mode (before cursor)
a          → Enter INSERT mode (after cursor)
A          → Enter INSERT mode at end of line
o          → Open new line below & enter INSERT mode
O          → Open new line above & enter INSERT mode
v          → Enter VISUAL mode (character select)
V          → Enter VISUAL LINE mode
Ctrl+v     → Enter VISUAL BLOCK mode
Esc        → Return to NORMAL mode
:          → Enter COMMAND mode

---

## NAVIGATION (NORMAL mode)
h          → Move left
l          → Move right
j          → Move down
k          → Move up
w          → Move to first letter of next word
b          → Move to first letter of previous word
e          → Move to last letter of current/next word
gg         → Move to top of file
G          → Move to bottom of file
0          → Move to start of line
^          → Move to first non-blank character of line
$          → Move to end of line
Ctrl+d     → Scroll down half a page
Ctrl+u     → Scroll up half a page
Ctrl+f     → Scroll down full page
Ctrl+b     → Scroll up full page
{          → Jump to previous empty line (paragraph up)
}          → Jump to next empty line (paragraph down)
%          → Jump to matching bracket/parenthesis

---

## EDITING (NORMAL mode)
x          → Delete character under cursor
dw         → Delete word under cursor
dd         → Delete (cut) current line
D          → Delete from cursor to end of line
d$         → Same as D
yy         → Yank (copy) current line
yw         → Yank current word
p          → Paste after cursor
P          → Paste before cursor
u          → Undo
Ctrl+r     → Redo
r          → Replace single character under cursor
cw         → Change (delete+insert) word under cursor
cc         → Change entire line
C          → Change from cursor to end of line
>>         → Indent line right
<<         → Indent line left
~          → Toggle case of character under cursor
.          → Repeat last action

---

## SELECT ALL / CLIPBOARD
ggVG       → Select all text
"*yy       → Yank line to system clipboard
:%y+       → Yank entire file to system clipboard
"*p        → Paste from system clipboard
"+y        → Yank selection to system clipboard (VISUAL mode)

---

## SEARCH & REPLACE
/pattern   → Search forward for pattern
?pattern   → Search backward for pattern
n          → Jump to next search match
N          → Jump to previous search match
:%s/old/new/g     → Replace all occurrences in file
:%s/old/new/gc    → Replace all with confirmation prompt

---

## FILE & COMMAND
:w         → Save file
:q         → Quit
:wq        → Save and quit
:q!        → Quit without saving
:e filename→ Open/edit a file
:set nu    → Show line numbers
:set nonu  → Hide line numbers
:%d        → Delete (clear) all lines in file

---

## DELETE IN INSERT MODE
Ctrl+w     → Delete previous word
Ctrl+u     → Delete entire line before cursor

---

## SPLITS & TABS
:split     → Horizontal split
:vsplit    → Vertical split
:vertterm  → Open a vertical terminal split
Ctrl+w w   → Switch between split panes
:tabnew    → Open new tab
gt         → Go to next tab
gT         → Go to previous tab