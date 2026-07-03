# Vim Cheatsheet

> Modal editing: NORMAL mode is a command language, not a waiting room. Covers Vim everywhere it matters (VPS boxes included) plus a Neovim section; same grammar, modern engine.

---

## Table of Contents
- [Modes](#modes)
- [The Grammar](#the-grammar)
- [Navigation — Basic](#navigation--basic)
- [Navigation — Lines & File](#navigation--lines--file)
- [Navigation — Advanced](#navigation--advanced)
- [Text Objects](#text-objects)
- [Editing — Delete](#editing--delete)
- [Editing — Change](#editing--change)
- [Editing — Copy & Paste](#editing--copy--paste)
- [Registers & Clipboard](#registers--clipboard)
- [Editing — Indent & Format](#editing--indent--format)
- [Visual Mode Operations](#visual-mode-operations)
- [Undo & Redo](#undo--redo)
- [Search & Replace](#search--replace)
- [Files & Buffers](#files--buffers)
- [Splits & Tabs](#splits--tabs)
- [Macros](#macros)
- [Marks & Jumps](#marks--jumps)
- [Line Operations](#line-operations)
- [Useful Commands](#useful-commands)
- [Common Patterns](#common-patterns)
- [Neovim](#neovim)
- [Tips & Gotchas](#tips--gotchas)

---

## MODES

```text
i        → Enter INSERT mode (before cursor)
I        → Enter INSERT mode (beginning of line)
a        → Enter INSERT mode (after cursor)
A        → Enter INSERT mode (end of line)
o        → Open new line below & enter INSERT
O        → Open new line above & enter INSERT
v        → Enter VISUAL mode (character select)
V        → Enter VISUAL mode (line select)
Ctrl+v   → Enter VISUAL BLOCK mode (column select)
R        → Enter REPLACE mode (overwrite text)
Esc      → Return to NORMAL mode
:        → Enter COMMAND mode
```

---

## THE GRAMMAR

Vim commands are sentences: **operator + count + motion/object**. Learn 5 operators and 10 motions, get 50 commands free.

```text
d2w      → delete 2 words          (operator d, count 2, motion w)
c$       → change to end of line
y}       → yank to next paragraph
>ip      → indent inner paragraph
gU aw    → uppercase a word

Operators: d delete · c change · y yank · > indent · < unindent
           = format · gu lowercase · gU uppercase
Doubling an operator applies it to the LINE: dd, cc, yy, >>, ==
```

---

## NAVIGATION — BASIC

```text
h        → Move left
j        → Move down
k        → Move up
l        → Move right
w        → Jump to start of next word
W        → Jump to start of next WORD (whitespace-delimited)
b        → Jump to start of previous word
B        → Jump to start of previous WORD
e        → Jump to end of current/next word
0        → Jump to beginning of line
^        → Jump to first non-blank character of line
$        → Jump to end of line
```

---

## NAVIGATION — LINES & FILE

```text
gg       → Go to first line of file
G        → Go to last line of file
:n       → Go to line n (e.g. :42)
nG       → Go to line n (e.g. 42G)
H        → Move cursor to top of screen
M        → Move cursor to middle of screen
L        → Move cursor to bottom of screen
Ctrl+u   → Scroll up half a page
Ctrl+d   → Scroll down half a page
Ctrl+b   → Scroll up full page
Ctrl+f   → Scroll down full page
zz       → Center current line on screen
zt       → Move current line to top of screen
zb       → Move current line to bottom of screen
```

---

## NAVIGATION — ADVANCED

```text
%        → Jump to matching bracket ( ) [ ] { }
f{char}  → Jump forward to {char} on current line
F{char}  → Jump backward to {char} on current line
t{char}  → Jump forward to just before {char}
T{char}  → Jump backward to just after {char}
;        → Repeat last f/F/t/T forward
,        → Repeat last f/F/t/T backward
*        → Jump to next occurrence of word under cursor
#        → Jump to previous occurrence of word under cursor
{        → Jump to previous empty line (paragraph up)
}        → Jump to next empty line (paragraph down)
Ctrl+o   → Jump to previous cursor position
Ctrl+i   → Jump to next cursor position
```

---

## TEXT OBJECTS

Work with **structures**, not character counts: `i` = inner (contents only), `a` = around (contents + delimiters). Combine with any operator — the cursor just needs to be INSIDE the object.

```text
iw / aw      → inner word / a word (with space)
i" / a"      → inside quotes / including quotes
i( or ib     → inside parentheses
i{ or iB     → inside braces
i[ / i<      → inside brackets / angle brackets
ip / ap      → inner paragraph / a paragraph
it / at      → inside / around HTML/XML tag

di(      → Delete everything inside the parentheses
ya"      → Yank the quoted string, quotes included
vip      → Visually select the paragraph
ci{      → Rewrite a code block's body
```

---

## EDITING — DELETE

```text
x        → Delete character under cursor
X        → Delete character before cursor
dw       → Delete from cursor to start of next word
db       → Delete from cursor to start of previous word
dd       → Delete entire line
D        → Delete from cursor to end of line
d$       → Delete from cursor to end of line (same as D)
d0       → Delete from cursor to beginning of line
d^       → Delete from cursor to first non-blank character
dG       → Delete from current line to end of file
dgg      → Delete from current line to beginning of file
5dd      → Delete 5 lines
```

---

## EDITING — CHANGE

```text
cw       → Change word (delete word & enter INSERT)
cc       → Change entire line
C        → Change from cursor to end of line
ci"      → Change inside double quotes
ci(      → Change inside parentheses
ci{      → Change inside curly braces
ci[      → Change inside square brackets
ciw      → Change inner word
caw      → Change a word (including surrounding space)
ct{char} → Change up to {char}
s        → Substitute character (delete & enter INSERT)
S        → Substitute entire line
r{char}  → Replace single character with {char}
~        → Toggle case of character under cursor
```

---

## EDITING — COPY & PASTE

```text
yy       → Yank (copy) entire line
yw       → Yank word
y$       → Yank from cursor to end of line
yG       → Yank from current line to end of file
p        → Paste after cursor
P        → Paste before cursor
xp       → Swap two characters
ddp      → Swap two lines
```

---

## REGISTERS & CLIPBOARD

Every delete/yank lands in a register — nothing is ever really lost.

```text
"ayy     → Yank line into register a
"ap      → Paste register a
"Ayy     → APPEND line to register a (capital = append)
"0p      → Paste the last YANK (ignores deletes that overwrote "")
:reg     → Inspect all registers
```

### System clipboard

```text
"+yy     → Yank line to system clipboard
"+p      → Paste from system clipboard
"*yy     → Yank to X11 primary selection (middle-click buffer)
"*p      → Paste from primary selection
:%y+     → Yank entire file to system clipboard
ggVG"+y  → Same, visually
```

```text
⚠️ "+/"*  need clipboard support: `vim --version | grep clipboard`
(-clipboard on Ubuntu's vim.tiny/basic → apt install vim-gtk3), and a
running X11/Wayland session. On a HEADLESS VPS neither exists — copy via
tmux (see tmux sheet) or Neovim ≥0.10's OSC 52 support (see Neovim).
```

---

## EDITING — INDENT & FORMAT

```text
>>       → Indent line right
<<       → Indent line left
>}       → Indent from cursor to next paragraph
<}       → Unindent from cursor to next paragraph
==       → Auto-indent current line
gg=G     → Auto-indent entire file
:set tabstop=4     → Set tab width to 4
:set expandtab     → Use spaces instead of tabs
:set shiftwidth=4  → Set indent width to 4
```

---

## VISUAL MODE OPERATIONS

```text
v        → Start character-wise selection
V        → Start line-wise selection
Ctrl+v   → Start block (column) selection
y        → Yank selected text
d        → Delete selected text
>        → Indent selected text
<        → Unindent selected text
=        → Auto-indent selection
U        → Uppercase selection
u        → Lowercase selection (yes — see Gotchas)
gv       → Reselect last visual selection
ggVG     → Select entire file
o        → Jump to other end of selection
```

### Block-edit multiple lines at once

```text
Ctrl+v → j j j → I  → type text → Esc   → Insert at start of every line
Ctrl+v → j j j → $ A → type text → Esc  → Append at end of every line
Ctrl+v → select column → d / r{char}    → Delete/replace the column
```

---

## UNDO & REDO

```text
u        → Undo last change
Ctrl+r   → Redo last undone change
U        → Undo all changes on current line
.        → Repeat last action
5.       → Repeat last action 5 times
```

---

## SEARCH & REPLACE

```text
/pattern → Search forward for pattern
?pattern → Search backward for pattern
n        → Jump to next match
N        → Jump to previous match
:noh     → Clear search highlighting
:s/old/new/      → Replace first on current line
:s/old/new/g     → Replace all on current line
:%s/old/new/g    → Replace all in entire file
:%s/old/new/gc   → Replace all with confirmation
:%s/old/new/gi   → Replace all case-insensitive
:%s//new/g       → Reuse the LAST SEARCH as the pattern
:'<,'>s/old/new/g → Replace within visual selection (range auto-fills)
```

---

## FILES & BUFFERS

```text
:w           → Save current file
:w filename  → Save as filename
:q           → Quit (fails if unsaved changes)
:q!          → Force quit (discard changes)
:wq          → Save and quit
:x           → Save (only if changed) and quit
ZZ           → Save and quit (normal mode shortcut)
ZQ           → Quit without saving (normal mode shortcut)
:e filename  → Open file in current buffer
:e!          → Reload current file (discard changes)
:bn          → Next buffer
:bp          → Previous buffer
:bd          → Close current buffer
:ls          → List all open buffers
:b name      → Jump to buffer by (partial) name
```

---

## SPLITS & TABS

```text
:split filename   → Horizontal split
:vsplit filename  → Vertical split
Ctrl+w s     → Horizontal split (current file)
Ctrl+w v     → Vertical split (current file)
Ctrl+w h     → Move to left split
Ctrl+w j     → Move to split below
Ctrl+w k     → Move to split above
Ctrl+w l     → Move to right split
Ctrl+w w     → Cycle through splits
Ctrl+w =     → Equal size all splits
Ctrl+w q     → Close current split
:tabnew filename  → Open file in new tab
gt           → Next tab
gT           → Previous tab
:tabclose    → Close current tab
```

---

## MACROS

```text
q{key}   → Start recording macro to register {key}
q        → Stop recording
@{key}   → Play macro from register {key}
@@       → Replay last played macro
5@{key}  → Play macro 5 times
:reg     → View all registers (including macros)
```

```text
💡 Macro discipline: start with a positioning motion (0 or ^), end by
moving to the NEXT target (j or n) — then 100@q chews through the file.
```

---

## MARKS & JUMPS

```text
m{key}   → Set mark at current position
'{key}   → Jump to line of mark
`{key}   → Jump to exact position of mark
:marks   → List all marks
''       → Jump to last position before jump
'.       → Jump to last edited line
```

```text
💡 Lowercase marks are per-file; CAPITAL marks (mA) work ACROSS files —
mark a config file once, 'A back to it from anywhere.
```

---

## LINE OPERATIONS

```text
J        → Join current line with next line
:sort    → Sort all lines alphabetically
:sort!   → Sort in reverse
:sort u  → Sort & remove duplicates
:%d      → Delete all lines in file
:g/pattern/d   → Delete all lines matching pattern
:v/pattern/d   → Delete all lines NOT matching pattern
Ctrl+a   → Increment number under cursor
Ctrl+x   → Decrement number under cursor
```

---

## USEFUL COMMANDS

```text
:set number          → Show line numbers
:set relativenumber  → Show relative line numbers
:set nonumber        → Hide line numbers
:set paste           → Paste mode (disables auto-indent)
:set nopaste         → Exit paste mode
:set ignorecase      → Case-insensitive search
:set smartcase       → Case-sensitive if uppercase present
:set hlsearch        → Highlight all search matches
:set incsearch       → Incremental search (highlight as you type)
:syntax on           → Enable syntax highlighting
:set wrap            → Enable line wrapping
:set nowrap          → Disable line wrapping
:set spell           → Enable spell checking
]s                   → Jump to next misspelled word
z=                   → Show spelling suggestions
```

---

## COMMON PATTERNS

```text
Ctrl+u       → Delete line before cursor in INSERT mode
Ctrl+w       → Delete previous word in INSERT mode
:!command    → Run shell command from vim
:r !command  → Insert output of shell command
:r filename  → Insert contents of file
gf           → Open file under cursor
vim +42 file → Open file AT line 42 (great from stack traces)
vim -d a b   → Diff two files (vimdiff)
```

```text
# Forgot sudo? Save anyway (vim only — see Neovim section):
:w !sudo tee % > /dev/null

# Edit a remote file over SSH (netrw — double slash = absolute path):
vim scp://vps1//etc/nginx/nginx.conf
```

---

## NEOVIM

Drop-in Vim successor: same grammar, everything above applies. Adds Lua config, built-in LSP/terminal, saner defaults (incsearch/hlsearch/syntax on out of the box).

```bash
sudo apt install neovim        # ⚠️ Ubuntu 22.04 ships ancient 0.6 — instead:
sudo add-apt-repository ppa:neovim-ppa/stable && sudo apt install neovim
nvim file.txt
echo "alias vim='nvim'" >> ~/.bashrc    # Muscle memory intact
```

### Minimal ~/.config/nvim/init.lua

```lua
vim.opt.number = true
vim.opt.relativenumber = true
vim.opt.expandtab = true
vim.opt.shiftwidth = 4
vim.opt.tabstop = 4
vim.opt.ignorecase = true
vim.opt.smartcase = true
vim.opt.undofile = true                 -- Persistent undo across sessions
vim.opt.clipboard = "unnamedplus"       -- y/p use the system clipboard
vim.g.mapleader = " "
vim.keymap.set("n", "<leader>w", ":w<CR>")
```

### Built-ins worth knowing

```text
:terminal            → Terminal inside the editor (Ctrl+\ Ctrl+n to escape to NORMAL)
:checkhealth         → Diagnose config/clipboard/provider issues
:checkhealth clipboard → Why "+y isn't working (needs xclip / wl-clipboard installed)
:Tutor               → Interactive tutorial
```

### Vim → Neovim differences that bite

```text
Config lives at ~/.config/nvim/init.lua (or init.vim) — not ~/.vimrc
:w !sudo tee %  does NOT work in nvim → use `sudoedit file` instead
Clipboard is a provider: install xclip (X11) or wl-clipboard (Wayland)
nvim ≥0.10 can use OSC 52 — system clipboard THROUGH SSH, no X needed
Bracketed paste is automatic — :set paste is never needed in nvim
```

```text
💡 Want LSP/autocomplete/fuzzy-finding without config archaeology:
start from kickstart.nvim (single commented init.lua) rather than a
mega-distribution — you'll understand every line you own.
```

---

## TIPS & GOTCHAS

- **`u` in VISUAL mode lowercases the selection** — it doesn't undo. Select-then-u is the classic "why is my code lowercase" accident; press `u` again *after* Esc to undo it.
- **Staircase indentation on paste** — terminal paste into INSERT mode replays autoindent. Legacy vim: `:set paste`, paste, `:set nopaste`. Modern vim/nvim handle bracketed paste automatically.
- **`"0p` recovers your yank** — deleted something after yanking and `p` gives the wrong text? The yank still lives in register 0.
- **`:x` vs `:wq`** — `:x` writes only if the buffer changed; `:wq` always writes (touches mtime, triggers file-watchers/rebuilds).
- **Arrow keys typing letters on a fresh server** — you're in `vi`/vim-tiny compatible mode. `sudo apt install vim` (or use nvim) and the pain ends.
- **Recording a macro by accident** — stray `q` starts recording (see `recording @q` in the status line); press `q` to stop, no harm done.
- **Swap file warnings after a crash/dropped SSH** — `vim -r file` recovers, then delete the `.swp`. Inside tmux (see tmux sheet) the session survives the disconnect and this never happens.
- **`.` is the cheapest macro** — structure edits as one repeatable change (`ciw` + text + Esc), then navigate (`n`, `;`, `j`) and `.` your way through the file.

---
*Last Updated: 2026-07*
