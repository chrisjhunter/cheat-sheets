# Git Workflows

Multi-step git procedures — the kind of thing you do a few times a year and
never quite remember. For quick one-liners, see [git.md](git.md).

## Comparing forks & fast-forwarding

Workflow for figuring out how two forks/branches relate before pushing, and
landing one cleanly onto the other.

```bash
git remote add theirfork <url>
git fetch theirfork

# Is A a strict ancestor of B? (i.e. B = A + more commits, no divergence)
git merge-base --is-ancestor theirfork/master mybranch && echo "clean fast-forward possible"

# Show exactly where two histories diverge
git merge-base theirfork/master mybranch  # the shared ancestor commit

# Side-by-side comparison of commit logs (diff -y style)
diff -y --width=180 <(git log --oneline theirfork/master) <(git log --oneline mybranch)

# Count commits unique to each side
git log --oneline theirfork/master..mybranch | wc -l  # on mybranch, not on theirs
git log --oneline mybranch..theirfork/master | wc -l  # on theirs, not on mybranch
```

If `--is-ancestor` says yes, a plain fast-forward push is safe (no merge
commit, no rewritten history):
```bash
git push theirfork mybranch:master
```

If it says no (real divergence — e.g. their branch moved forward on its own
after you branched off it), merge their tip in first, then the push above
becomes a fast-forward:
```bash
git merge theirfork/master -m "Merge theirfork/master into mybranch"
git merge-base --is-ancestor theirfork/master mybranch && git push theirfork mybranch:master
```

Standard origin/upstream convention when you're working on someone else's
project via your own fork (origin = your fork you push to, upstream = the
project you pull updates from):
```bash
git remote rename origin upstream  # if it's currently pointed at the source project
git remote rename myfork origin    # point origin at your own fork instead
git remote -v                      # confirm
```
