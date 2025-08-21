# Git CLI commands

## Setup

```bash
git --version
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global core.autocrlf input      # Windows: true
git config --global pull.rebase false        # or true (team preference)
git config --global color.ui auto
git config --global core.editor "code --wait"
git config --list --show-origin
```

### SSH auth (recommended)

```bash
ssh-keygen -t ed25519 -C "you@example.com"
cat ~/.ssh/id_ed25519.pub     # add to Git host
ssh -T git@github.com         # test
```

## Create / Clone

```bash
git init
git init my-repo
git clone <url>
git clone --depth 1 <url>                     # shallow
git clone --filter=blob:none <url>            # partial clone (faster)
git remote -v
git remote add origin <url>
```

## Status & Stage

```bash
git status
git add .
git add -p                                     # patch/interactive
git restore --staged <path>                    # unstage
git restore <path>                             # discard unstaged edits
git rm <path>                                  # remove + track deletion
git mv <old> <new>
```

## Commit

```bash
git commit -m "feat: add X"
git commit -am "fix: quick edit"               # stage tracked + commit
git commit --amend                             # edit last commit
git commit --amend --no-edit
```

## Branching & Switching

```bash
git branch
git branch -vv
git switch -c feature/login
git switch main
# (older) git checkout -b feature/login; git checkout main
```

## Sync with Remote

```bash
git fetch --all --prune
git pull
git pull --rebase
git push -u origin <branch>                    # first push sets upstream
git push
git push --force-with-lease                    # safe force
```

## Merge / Rebase / Cherry-pick

```bash
git merge feature/login
git rebase main
git rebase -i HEAD~5                           # interactive: squash/fixup
git cherry-pick <commit>
git revert <commit>                            # undo by new commit
```

Tip: teams often prefer `--no-ff` merges to keep feature history.

## Inspect History & Diff

```bash
git log --oneline --graph --decorate --all
git log -p <path>
git log --stat -- <path>
git show <commit>
git diff                                       # WT vs index
git diff --staged                              # index vs HEAD
git blame <path>
```

## Stash (shelve)

```bash
git stash push -m "wip: search form"
git stash push -u -m "wip"                     # include untracked
git stash list
git stash show -p stash@{0}
git stash apply stash@{0}
git stash pop
git stash drop stash@{0}
```

## Undo & Repair

```bash
git restore <path>                             # discard unstaged
git restore --staged <path>                    # unstage
git reset --soft HEAD~1                        # keep staged
git reset --mixed HEAD~1                       # keep working tree
git reset --hard HEAD~1                        # ⚠️ lose local changes
git reflog                                     # find lost commits/moves
```

## Tags & Releases

```bash
git tag
git tag v1.2.0
git tag -a v1.2.0 -m "Release 1.2.0"
git show v1.2.0
git push origin v1.2.0
git push origin --tags
git tag -d v1.2.0
git push origin :refs/tags/v1.2.0
```

## Worktrees (multiple checkouts)

```bash
git worktree add ../proj-main main
git worktree list
git worktree remove ../proj-main
```

## Sparse Checkout (partial working tree)

```bash
git sparse-checkout init --cone
git sparse-checkout set src apps/web docs
```

## Submodules

```bash
git submodule add <url> vendor/lib
git submodule update --init --recursive
git submodule foreach git pull --rebase
```

## Search & Grep

```bash
git grep "TODO"
git grep -n "myFunc" -- '*.cs'
```

## Archive / Export

```bash
git archive -o release.zip HEAD
git archive -o src.zip v1.2.0
```

## Bisect (find bad commit)

```bash
git bisect start
git bisect bad
git bisect good v1.1.0
# test, then mark bad|good repeatedly
git bisect reset
```

## Signing (GPG/Sigstore)

```bash
git config --global commit.gpgsign true
git config --global user.signingkey <KEYID>
git commit -S -m "feat: signed commit"
git tag -s v1.3.0 -m "signed tag"
```

## Ignore & Attributes

```bash
echo "node_modules/" >> .gitignore
git rm -r --cached node_modules/
git update-index --skip-worktree <path>        # keep local edits; ignore merges
git update-index --no-skip-worktree <path>
# .gitattributes examples:
echo "* text=auto eol=lf" >> .gitattributes
echo "*.png binary" >> .gitattributes
```

## Handy Aliases (`~/.gitconfig`)

```ini
[alias]
  co = checkout
  sw = switch
  br = branch -vv
  st = status -sb
  lg = log --oneline --graph --decorate --all
  last = log -1 HEAD
  aa = add -A
  ca = commit -a
  cm = commit -m
  amend = commit --amend --no-edit
  df = diff
  dfs = diff --staged
  fp = fetch --prune --all
  pv = push -v
  rv = remote -v
```

## Everyday Recipes

```bash
# Start a feature
git switch -c feature/calc
git add -p && git commit -m "feat(calc): initial"

# Keep up with main
git fetch origin
git rebase origin/main     # or: git merge origin/main

# Fix last commit
git commit --amend

# Squash before push
git rebase -i origin/main

# Resolve conflicts and continue rebase
git add <files>
git rebase --continue

# Push safely
git push -u origin feature/calc
git push --force-with-lease   # only after rebase
```

## Troubleshooting

* Detached HEAD → `git switch -c temp` then merge/cherry-pick as needed.
* Abort a merge before commit → `git merge --abort`
* Forgot to stage a file → `git add <file> && git commit --amend`
* Lost work? → `git reflog` then `git switch <sha>` or `git cherry-pick <sha>`
* Slow huge repo? → `git clone --filter=blob:none` + `git sparse-checkout`

---
