# Git Cheat Sheet

## Setup
```bash
git config --global user.name "Name"
git config --global user.email "email@example.com"
git config --global init.defaultBranch main
git config --global alias.co checkout
```

## Basics
```bash
git init
git clone <url>
git status
git add -p                    # stage interactively, hunk by hunk
git commit -m "msg"
git commit --amend --no-edit  # fix last commit without changing message
```

## Branching
```bash
git branch                 # list local branches
git branch -a              # list all (incl. remote)
git checkout -b feature/x  # create + switch
git switch -c feature/x    # modern equivalent
git branch -d feature/x    # delete (safe)
git branch -D feature/x    # delete (force)
git push origin --delete feature/x
```

## Inspecting
```bash
git log --oneline --graph --all
git log -p -- file.txt    # patch history for a file
git log --author="chris"
git blame file.txt
git diff                  # unstaged changes
git diff --staged         # staged changes
git diff main..feature/x  # compare branches
git show <commit>
```

## Undo & rewrite history
```bash
git restore file.txt           # discard unstaged changes
git restore --staged file.txt  # unstage
git reset --soft HEAD~1        # undo commit, keep changes staged
git reset --mixed HEAD~1       # undo commit, unstage changes (default)
git reset --hard HEAD~1        # undo commit, discard changes (destructive)
git revert <commit>            # new commit that undoes a commit (safe for shared history)
git rebase -i HEAD~5           # interactive rebase last 5 commits
git cherry-pick <commit>
git reflog                     # recover "lost" commits
```

## Stash
```bash
git stash                  # stash tracked changes
git stash -u               # include untracked files
git stash list
git stash pop              # apply + drop most recent
git stash apply stash@{1}  # apply specific stash without dropping
git stash drop stash@{0}
```

## Remotes & syncing
```bash
git remote -v
git remote add origin <url>
git fetch --all --prune
git pull --rebase
git push -u origin main
git push --force-with-lease  # safer force push (checks remote hasn't changed)
```

> For multi-step procedures (fork comparison, fast-forwarding, upstream/origin setup), see [git-workflows.md](git-workflows.md).

## Tags
```bash
git tag v1.0.0
git tag -a v1.0.0 -m "release"
git push origin v1.0.0
git push origin --tags
```

## Useful one-liners
```bash
git log --oneline | wc -l                                      # commit count
git shortlog -sn                                               # commits per author
git diff --stat main..HEAD                                     # summary of changed files vs main
git ls-files | xargs wc -l | tail -1                           # total lines tracked
git clean -fd                                                  # remove untracked files/dirs (destructive)
git branch --merged main | grep -v main | xargs git branch -d  # clean merged branches
git log --diff-filter=D --summary                              # find deleted files
git grep "TODO"                                                # search tracked files
git bisect start; git bisect bad; git bisect good v1.0         # binary search for bug
git worktree add ../hotfix main                                # separate working dir for another branch
```
