# Temp Files & Scratch Space Cheat Sheet

Patterns for working in `/tmp` (or an equivalent scratch dir) safely —
why `mktemp` instead of a hardcoded path, how to guarantee cleanup, and
where else temp state tends to live.

## mktemp basics
```bash
mktemp                       # create + print path to a unique temp file
mktemp -d                    # create + print path to a unique temp directory
mktemp /tmp/myapp.XXXXXX     # custom prefix, XXXXXX gets randomized
mktemp -d /tmp/myapp.XXXXXX  # same, for a directory
mktemp --suffix=.log         # keep a meaningful extension
mktemp -p /var/tmp           # use a specific base dir instead of $TMPDIR
```
**Why `mktemp` over `/tmp/myfile` or `/tmp/$$`:** a fixed or PID-based name is
predictable. On a shared multi-user box another process (or an attacker)
can pre-create that path as a symlink pointing somewhere you don't intend
to write, or simply collide with a concurrent run of your own script.
`mktemp` atomically creates the file/dir with a random suffix and mode
`0600`/`0700`, so there's no window where the name exists but isn't yours.

## Capturing the path safely
```bash
tmpfile=$(mktemp)  # capture into a variable, use it everywhere after
tmpdir=$(mktemp -d)
echo "working in $tmpdir"
```
**Why:** always assign to a variable rather than re-running `mktemp` or
re-deriving the path — every call produces a *new* unique name, so calling
it twice gives you two different files, not the one you just made.

## Guaranteed cleanup with trap
```bash
tmpdir=$(mktemp -d)
trap 'rm -rf "$tmpdir"' EXIT  # runs on normal exit AND on error/interrupt
```
**Why:** `trap ... EXIT` fires whether the script finishes normally, hits
`set -e` and bails, or gets Ctrl-C'd — a plain `rm -rf "$tmpdir"` at the
bottom of the script only runs on the happy path and leaks scratch dirs
on every early exit. This is the single most important habit here.

```bash
trap 'rm -rf "$tmpdir"' EXIT INT TERM  # explicit about which signals, if EXIT alone feels too magic
```

## Isolating a whole task under one scratch dir
```bash
workdir=$(mktemp -d -t myscript-XXXXXX)
cd "$workdir"
# ...download, generate, process...
cd - >/dev/null
rm -rf "$workdir"
```
**Why one dir instead of many loose temp files:** everything the task
produces lives under a single throwaway root, so cleanup is one `rm -rf`,
nothing can collide with a concurrent run, and if something goes wrong
mid-task you can inspect the whole workdir before it's cleaned up
(comment out the trap temporarily to debug).

## Session-scoped / long-lived scratch dirs
```bash
mkdir -p "/tmp/myproject-$USER"                # per-user, per-project, survives one login session
mkdir -p "${XDG_RUNTIME_DIR:-/tmp}/myproject"  # prefer XDG_RUNTIME_DIR when available
```
**Why `$XDG_RUNTIME_DIR` over `/tmp` when it exists:** it's per-user,
tmpfs-backed, mode `0700`, and cleaned up automatically at logout by
systemd — no manual cleanup logic needed, and no risk of another user on
the box reading your files (unlike world-readable `/tmp`).

```bash
mkdir -p "/tmp/myapp-$USER/$$"  # namespace by user AND a unique run id ($$ = this shell's PID)
```
**Why namespace by both user and a run id:** a shared prefix like
`/tmp/myapp-$USER/` keeps a project's scratch files grouped and easy to
find/clean as a whole, while a per-run subdirectory (PID, session UUID, or
timestamp) means two concurrent runs of the same tool never step on each
other's intermediate files — this is the same idea behind Claude Code's
own `/tmp/claude-<uid>/<project>/<session-id>/scratchpad/` convention:
project-scoped, session-scoped, and disposable.

## Dump-edit-load: safely editing structured system state
```bash
crontab -l > /tmp/cron_fix.txt  # dump current state to an editable file
vim /tmp/cron_fix.txt           # edit with a real editor, not a one-line sed guess
crontab /tmp/cron_fix.txt       # load the edited version back in
```
**Why not edit in place with `sed -i` on the live crontab:** `crontab -e`
and direct edits give you no diff, no way to review the change before it
takes effect, and no easy rollback. Round-tripping through a plain temp
file lets you `diff` before/after, works the same way for any tool that
reads/writes structured config from stdin (`crontab`, `visudo`-style
edits, k8s `kubectl edit` under the hood), and leaves a paper trail of
exactly what changed if you keep the file around.

## Backup before a destructive/risky edit
```bash
cp go.mod /tmp/go.mod.bak; cp go.sum /tmp/go.sum.bak  # snapshot before a risky dependency change
# ...run go mod tidy, go get -u, etc...
diff /tmp/go.mod.bak go.mod                           # confirm what actually changed
cp /tmp/go.mod.bak go.mod                             # one-line revert if it went sideways
```
**Why a `/tmp` copy instead of relying on git:** even in a git repo, a
quick `/tmp/*.bak` copy is a zero-ceremony rollback for a single risky
command — no `git stash`/`git checkout` ceremony, no risk of catching
unrelated unstaged changes in the same revert, and it works identically
in directories that aren't under version control at all.

## Atomic writes (don't let readers see a half-written file)
```bash
mktemp_file=$(mktemp)
generate_report > "$mktemp_file"
mv "$mktemp_file" report.txt  # mv within the same filesystem is atomic
```
**Why:** writing directly to the destination path means anything reading
it mid-write sees a truncated/partial file. Writing to a temp file first
and `mv`-ing it into place is atomic on the same filesystem — readers see
either the old complete file or the new complete file, never a partial
one. (Cross-filesystem `mv` falls back to copy+delete and loses this
guarantee — keep the temp file on the same filesystem as the target.)

## Downloading/processing before trusting the result
```bash
tmpfile=$(mktemp)
curl -fsSL https://example.com/installer.sh -o "$tmpfile"
sha256sum -c expected.sha256 <<< "$(cat expected_hash)  $tmpfile"
bash "$tmpfile"
rm -f "$tmpfile"
```
**Why not pipe straight to `bash`:** `curl ... | bash` runs code you never
got to inspect or checksum, and if the download is truncated mid-stream
you can end up executing a half-written script. Downloading to a temp
file first gives you a checkpoint to verify before execution.

## Picking where scratch space lives
```bash
echo $TMPDIR       # respected by mktemp and many tools; often unset on Linux
mount | grep /tmp  # check if /tmp is tmpfs (RAM-backed) or disk-backed
df -h /tmp
```
**Why it matters:** tmpfs-backed `/tmp` is fast but counts against RAM and
disappears on reboot — fine for scratch work, bad for anything you need
to survive a restart or that's large enough to pressure memory. Use
`/var/tmp` (disk-backed, typically survives reboots longer) for bigger or
longer-lived intermediates.

## Cleaning up stale temp files
```bash
find /tmp -maxdepth 1 -user "$USER" -mtime +7 -exec rm -rf {} +  # your own files older than 7 days
sudo systemctl status systemd-tmpfiles-clean.timer               # systemd's own periodic /tmp sweep
sudo systemd-tmpfiles --clean                                    # trigger it manually
cat /usr/lib/tmpfiles.d/tmp.conf                                 # see the age rules systemd applies
```
**Why not just `rm -rf /tmp/*`:** other users' and other processes'
(possibly still-open) files live there too. Scope any manual cleanup to
your own user and a safe age threshold, and prefer letting
`systemd-tmpfiles` (or `tmpwatch`/`tmpreaper` on older distros) handle the
general sweep — it already knows which paths are safe to age out.

## Useful one-liners
```bash
mktemp -d && cd "$_"                                   # jump straight into a new scratch dir
tmpdir=$(mktemp -d); trap 'rm -rf "$tmpdir"' EXIT      # the whole safe pattern in one line
du -sh /tmp/* 2>/dev/null | sort -rh | head            # what's eating space in /tmp right now
lsof +D /tmp 2>/dev/null | awk '{print $2}' | sort -u  # PIDs currently holding files open under /tmp
find /tmp -maxdepth 1 -user "$USER"                    # everything of yours currently in /tmp
```
