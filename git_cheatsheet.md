# Git Cheatsheet

> Version control: stage, commit, branch, merge, and most importantly...undo. Nothing committed is ever truly lost (see reflog); most disasters are one command from recovery.

---

## Table of Contents
- [Setup & Config](#setup--config)
- [Init & Clone](#init--clone)
- [Status & Diff](#status--diff)
- [Staging & Committing](#staging--committing)
- [Branching](#branching)
- [Merging & Rebasing](#merging--rebasing)
- [Remote](#remote)
- [Undoing](#undoing)
- [Stashing](#stashing)
- [Tags](#tags)
- [Inspection & History](#inspection--history)
- [.gitignore](#gitignore)
- [Aliases](#aliases)
- [Tips & Gotchas](#tips--gotchas)

---

## SETUP & CONFIG

```bash
git config --global user.name "Your Name"
git config --global user.email "you@email.com"
git config --global init.defaultBranch main    # New repos start on main
git config --global pull.rebase true           # Pull = fetch + rebase (linear history)
git config --global core.editor "nvim"
git config --list                              # View all config settings
git config --global --edit                     # Edit config directly
```

```text
💡 Commit signing (GPG or SSH key) lives in the SSH/GPG Keys sheet.
```

---

## INIT & CLONE

```bash
git init                                   # Init new repo in current dir
git clone https://github.com/user/repo     # Clone remote repo
git clone git@github.com:user/repo.git     # Clone over SSH (see SSH sheet)
git clone repo ./folder                    # Clone into specific folder
git clone --depth 1 URL                    # Shallow clone (latest snapshot only)
```

---

## STATUS & DIFF

```bash
git status                                 # Working tree status
git status -s                              # Short format
git diff                                   # Unstaged changes
git diff --staged                          # Staged changes
git diff main..feature                     # Between branches
git diff HEAD~2 -- file.txt                # File vs 2 commits ago
git show commit_hash                       # Details of a commit
```

---

## STAGING & COMMITTING

```bash
git add file.txt                           # Stage specific file
git add .                                  # Stage all changes
git add -p                                 # Stage changes interactively (by hunk)
git commit -m "message"                    # Commit with message
git commit -am "message"                   # Stage tracked files + commit
git commit --amend -m "new message"        # Edit last commit message
git commit --amend --no-edit               # Add staged changes to last commit
```

---

## BRANCHING

```bash
git branch                                 # List local branches
git branch -a                              # List all branches (inc. remote)
git branch feature-name                    # Create new branch
git checkout feature-name                  # Switch to branch
git checkout -b feature-name               # Create & switch in one step
git switch feature-name                    # Modern switch syntax
git switch -c feature-name                 # Modern create & switch
git switch -                               # Back to previous branch
git branch -d feature-name                 # Delete branch (safe — merged only)
git branch -D feature-name                 # Force delete branch
git branch -m old-name new-name            # Rename branch
```

---

## MERGING & REBASING

```bash
git merge feature-name                     # Merge branch into current
git merge --no-ff feature-name             # Merge with merge commit
git merge --squash feature-name            # Squash into one commit
git merge --abort                          # Bail out of a conflicted merge
git rebase main                            # Rebase current onto main
git rebase -i HEAD~3                       # Interactive rebase (last 3 commits)
git rebase --abort                         # Bail out of a conflicted rebase
git cherry-pick commit_hash                # Apply specific commit to current branch
```

### Conflict resolution flow

```bash
git status                    # Which files conflict
# edit files — resolve <<<<<<< ======= >>>>>>> markers
git add resolved-file.txt
git merge --continue          # (or git rebase --continue)
```

---

## REMOTE

```bash
git remote -v                              # List remotes
git remote add origin URL                  # Add remote
git remote set-url origin URL              # Change remote URL (e.g. HTTPS → SSH)
git fetch origin                           # Fetch without merging
git fetch --prune                          # Also drop deleted remote branches
git pull origin main                       # Fetch & merge
git pull --rebase origin main              # Fetch & rebase
git push origin main                       # Push to remote
git push -u origin feature-name            # Push new branch & track
git push --force-with-lease                # Safe force push (see Gotchas)
git push origin --delete branch-name       # Delete remote branch
```

---

## UNDOING

```bash
git restore file.txt                       # Discard unstaged changes
git restore --staged file.txt              # Unstage a file
git restore --source HEAD~2 file.txt       # File as it was 2 commits ago
git reset --soft HEAD~1                    # Undo commit, keep staged
git reset --mixed HEAD~1                   # Undo commit, keep unstaged (default)
git reset --hard HEAD~1                    # Undo commit, DISCARD changes
git revert commit_hash                     # New commit that undoes a commit (safe on pushed history)
git clean -nd                              # PREVIEW untracked file removal
git clean -fd                              # Remove untracked files & dirs (irreversible)
```

### reflog — the undo for your undo

```bash
git reflog                                 # Every HEAD position, even "lost" ones
git reset --hard HEAD@{2}                  # Jump back to where you were 2 moves ago
git branch rescue HEAD@{5}                 # Resurrect lost commits onto a branch
```

---

## STASHING

```bash
git stash                                  # Stash current changes
git stash -u                               # Include untracked files
git stash push -m "description"            # Stash with label
git stash list                             # List all stashes
git stash pop                              # Apply latest stash & remove it
git stash apply stash@{1}                  # Apply specific stash (keep it)
git stash drop stash@{1}                   # Delete specific stash
git stash clear                            # Remove all stashes
```

---

## TAGS

```bash
git tag                                    # List tags
git tag v1.0.0                             # Create lightweight tag
git tag -a v1.0.0 -m "release"             # Create annotated tag (preferred for releases)
git push origin v1.0.0                     # Push specific tag
git push origin --tags                     # Push all tags
git tag -d v1.0.0                          # Delete local tag
git push origin --delete v1.0.0            # Delete remote tag
```

---

## INSPECTION & HISTORY

```bash
git log                                    # Full commit history
git log --oneline                          # Compact commit history
git log --oneline --graph --all            # Visual branch graph
git log -p file.txt                        # History WITH diffs for a file
git log --follow file.txt                  # History across renames
git log -S "function_name"                 # Commits that added/removed a string (pickaxe)
git log --since="2 weeks ago" --author=jedi
git blame file.txt                         # Who changed each line
git shortlog -sn                           # Commit count per author
```

### bisect — binary-search the commit that broke it

```bash
git bisect start
git bisect bad                             # Current commit is broken
git bisect good v1.0.0                     # This one was fine
# git checks out the midpoint — test it, then:
git bisect good   # or: git bisect bad     # repeat until culprit found
git bisect reset                           # Return to where you started
```

---

## .GITIGNORE

```bash
# Common patterns:
.env                  # Secrets — never commit
.venv/
__pycache__/
*.pyc
node_modules/
*.log
.DS_Store
```

```bash
git rm --cached file.txt        # Untrack an already-committed file (keeps it on disk)
git check-ignore -v file.txt    # WHY is this file ignored?
git config --global core.excludesFile ~/.gitignore_global   # Personal global ignores
```

---

## ALIASES

```bash
git config --global alias.st "status -s"
git config --global alias.lg "log --oneline --graph --all"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "restore --staged"
# Usage: git st, git lg, git last, git unstage file.txt
```

---

## TIPS & GOTCHAS

- **`reset --hard` is recoverable, `clean -fd` is not** — reset only moves pointers; `git reflog` gets committed work back. `clean` deletes untracked files that git never saw — run `clean -nd` (dry run) first, always.
- **`--force-with-lease` over `--force`** — it refuses to push if someone else pushed since you last fetched; plain `--force` silently erases their work.
- **Never amend/rebase pushed commits** on shared branches — you're rewriting history others have built on. Pushed and wrong? `git revert`.
- **"Detached HEAD" isn't broken** — you checked out a commit instead of a branch. Poke around freely; to keep work made there: `git switch -c new-branch`.
- **Committed to the wrong branch?** — `git switch correct-branch && git cherry-pick <hash>`, then remove it from the wrong branch with `git reset --hard HEAD~1`.
- **`.env` in history is compromised even after deletion** — removing it in a new commit doesn't remove it from history. Rotate the secrets; scrub history only if you must (git-filter-repo), knowing it rewrites every hash.
- **Prefer SSH remotes** — `git@github.com:user/repo.git` with the key + agent setup from the SSH/Keys sheets beats HTTPS token management.
- **`pull.rebase true` keeps history linear** — no more "Merge branch 'main' of github.com..." noise commits from routine pulls.

---
*Last Updated: 2026-07*
