# git_cheatsheet

# SETUP

---

git config --global [user.name](http://user.name/) "Your Name"
git config --global user.email "[you@email.com](mailto:you@email.com)"
git config --list                          → View all config settings

---

## INIT & CLONE

git init                                   → Init new repo in current dir
git clone [https://github.com/user/repo](https://github.com/user/repo)    → Clone remote repo
git clone repo ./folder                   → Clone into specific folder

---

## STATUS & DIFF

git status                                 → Working tree status
git diff                                   → Unstaged changes
git diff --staged                          → Staged changes
git log                                    → Full commit history
git log --oneline                          → Compact commit history
git log --oneline --graph --all            → Visual branch graph
git show commit_hash                       → Details of a commit

---

## STAGING & COMMITTING

git add file.txt                           → Stage specific file
git add .                                  → Stage all changes
git add -p                                 → Stage changes interactively (by hunk)
git commit -m "message"                    → Commit with message
git commit --amend -m "new message"        → Edit last commit message
git commit --amend --no-edit               → Add staged changes to last commit

---

## BRANCHING

git branch                                 → List local branches
git branch -a                              → List all branches (inc. remote)
git branch feature-name                    → Create new branch
git checkout feature-name                  → Switch to branch
git checkout -b feature-name               → Create & switch in one step
git switch feature-name                    → Modern switch syntax
git switch -c feature-name                 → Modern create & switch
git branch -d feature-name                 → Delete branch (safe)
git branch -D feature-name                 → Force delete branch
git branch -m old-name new-name            → Rename branch

---

## MERGING & REBASING

git merge feature-name                     → Merge branch into current
git merge --no-ff feature-name             → Merge with merge commit
git merge --squash feature-name            → Squash into one commit
git rebase main                            → Rebase current onto main
git rebase -i HEAD~3                       → Interactive rebase (last 3 commits)
git cherry-pick commit_hash                → Apply specific commit to current branch

---

## REMOTE

git remote -v                              → List remotes
git remote add origin URL                  → Add remote
git remote set-url origin URL             → Change remote URL
git fetch origin                           → Fetch without merging
git pull origin main                       → Fetch & merge
git pull --rebase origin main             → Fetch & rebase
git push origin main                       → Push to remote
git push -u origin feature-name           → Push new branch & track
git push --force-with-lease               → Safe force push
git push origin --delete branch-name      → Delete remote branch

---

## UNDOING

git restore file.txt                       → Discard unstaged changes
git restore --staged file.txt             → Unstage a file
git reset --soft HEAD~1                   → Undo commit, keep staged
git reset --mixed HEAD~1                  → Undo commit, keep unstaged
git reset --hard HEAD~1                   → Undo commit, discard changes
git revert commit_hash                    → New commit that undoes a commit
git clean -fd                             → Remove untracked files & dirs

---

## STASHING

git stash                                  → Stash current changes
git stash push -m "description"           → Stash with label
git stash list                             → List all stashes
git stash pop                              → Apply latest stash & remove it
git stash apply stash@{1}                 → Apply specific stash (keep it)
git stash drop stash@{1}                  → Delete specific stash
git stash clear                            → Remove all stashes

---

## TAGS

git tag                                    → List tags
git tag v1.0.0                            → Create lightweight tag
git tag -a v1.0.0 -m "release"           → Create annotated tag
git push origin v1.0.0                    → Push specific tag
git push origin --tags                    → Push all tags

---

## USEFUL

git blame file.txt                         → See who changed each line
git bisect start                           → Start binary search for bug
git shortlog -sn                           → Commit count per author
.gitignore                                 → File to exclude from tracking