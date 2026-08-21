# Resource Control Cheat Sheet

Keeping one process from starving everything else on the box, using kernel
scheduling knobs instead of the app's own flags. Grew out of running a
batch job (Overviewer map renders) alongside a live Minecraft server on a
4-core box — the fix wasn't a renderer setting, it was `nice`/`ionice`.

## The quick combo
```bash
nice -n 19 ionice -c 3 cmd args...    # lowest CPU priority + idle I/O class
```
This is usually enough: cap CPU niceness to the max (19, lowest priority)
and I/O to the "idle" class (only gets disk time when nothing else wants
it). Put it in front of any batch/background job that runs alongside
something latency-sensitive (a game server, a database, a web server).

## nice / renice — CPU scheduling priority
```bash
nice -n 19 cmd                # start a new process at lowest priority (-20 highest, 19 lowest)
nice -n 10 cmd                 # milder deprioritization
renice -n 15 -p 1234           # change priority of an already-running PID
renice -n 15 -g 5678            # by process group
ps -o pid,ni,cmd -p 1234        # check a process's current niceness (NI column)
```
Only affects CPU scheduling — a niced process still competes fully for
memory and disk I/O unless you also handle those (see below). Regular
users can only raise niceness (deprioritize themselves), not lower it,
without extra privileges.

## ionice — disk I/O scheduling class
```bash
ionice -c 3 cmd                     # idle class: only runs when disk is otherwise free
ionice -c 2 -n 7 cmd                  # best-effort class, lowest priority within it (0-7)
ionice -c 1 -n 0 cmd                   # realtime class, highest prio (needs root, rarely appropriate)
ionice -p 1234                          # check a running process's I/O class
ionice -c 3 -p 1234                      # apply idle I/O class to an already-running PID
```
Only has real effect under I/O schedulers that honor it (CFQ historically;
on modern kernels with `mq-deadline`/`bfq`, effect varies — `bfq` respects
it reasonably well, `mq-deadline` mostly ignores priority). Check the
current scheduler:
```bash
cat /sys/block/sda/queue/scheduler   # bracketed value is active
```

## cgroups v2 — hard resource caps, not just scheduling hints
`nice`/`ionice` are *hints* the scheduler can still deprioritize-but-allow;
cgroups let you put a hard ceiling on CPU %, memory, or I/O bandwidth.
`systemd-run` is the easy front door to cgroups on any systemd system:
```bash
# Cap a one-off command to 50% of one core and 512M memory
systemd-run --scope -p CPUQuota=50% -p MemoryMax=512M cmd args...

# Same, but also throttle disk I/O bandwidth on a specific device
systemd-run --scope -p CPUQuota=50% -p IOReadBandwidthMax="/dev/sda 10M" cmd

# Deprioritize I/O relative to everything else instead of a hard cap
systemd-run --scope -p IOWeight=10 cmd    # 1-10000, default 100

# Inspect what's actually applied
systemctl status run-<id>.scope
systemd-cgtop                              # live view of cgroup resource usage
```
For a long-running service instead of a one-off command, the same
properties go in the unit file (`CPUQuota=`, `MemoryMax=`, `IOWeight=`
under `[Service]`), or via `systemctl set-property <unit> CPUQuota=50%`
for a live change without editing the file.

## taskset / cpuset — CPU affinity (pin, don't throttle)
Doesn't limit *how much* CPU a process uses, just *which* cores it's
allowed to run on — useful to keep a batch job off the cores your
latency-sensitive process depends on, on a multi-core box.
```bash
taskset -c 2,3 cmd                  # run only on cores 2 and 3
taskset -cp 0,1 1234                 # restrict an existing PID to cores 0-1
taskset -cp 1234                      # check current affinity
nproc                                  # how many cores you actually have
```

## cpulimit — throttle an already-running process's CPU %
```bash
cpulimit -p 1234 -l 50               # cap PID 1234 to 50% of one core
cpulimit -e java -l 200                # cap by process name instead of PID
```
Different from `nice`: this enforces an actual ceiling (via SIGSTOP/SIGCONT
cycling) rather than just deprioritizing under contention — the process
gets throttled even if the CPU is otherwise idle. Not installed by
default on most distros (`apt install cpulimit` / `brew install cpulimit`).

## ulimit / prlimit — per-process resource limits
```bash
ulimit -a                          # show all current limits for this shell
ulimit -v 2097152                   # cap virtual memory to ~2GB (KB) for this shell + children
ulimit -n 4096                       # raise open-file-descriptor limit
prlimit --pid 1234                    # show limits for a running process
prlimit --pid 1234 --as=2147483648     # set address-space (memory) limit on a running PID
```
`ulimit` only affects the current shell and anything it spawns afterward —
set it right before launching the command, in the same shell/script.

## timeout — bound how long something is allowed to run
```bash
timeout 300 cmd                    # kill after 5 minutes if still running
timeout -s SIGKILL 60 cmd           # force-kill (not just SIGTERM) after 60s
timeout --foreground 300 cmd         # needed if cmd reads from the terminal
```
Good insurance for any batch job with no natural end condition, so a
stuck run can't monopolize resources indefinitely.

## Bandwidth throttling (network I/O)
```bash
trickle -d 500 -u 100 cmd            # cap download to 500KB/s, upload to 100KB/s (per-app, userspace)
wondershaper eth0 5000 1000            # cap an interface: 5000kbit/s down, 1000kbit/s up (system-wide)
```
`trickle` wraps a single command; `wondershaper`/`tc` shape a whole
interface. Neither is installed by default on most distros.

## Containers: the equivalent flags
If the noisy-neighbor process is containerized instead of a bare process:
```bash
docker run --cpus=1.5 --memory=512m --blkio-weight=50 image
podman run --cpus=1.5 --memory=512m image
```
Docker/Podman set up the same cgroups under the hood — these are just a
friendlier front end for the containerized case.

## Picking the right tool
| Need | Tool |
|---|---|
| Deprioritize CPU under contention, no hard cap | `nice` |
| Deprioritize disk I/O under contention | `ionice` |
| Hard CPU/memory/IO ceiling | `systemd-run` (cgroups) |
| Keep a job off specific cores | `taskset` |
| Hard CPU % cap on an existing PID, even when idle | `cpulimit` |
| Cap memory/fd limits for a shell session | `ulimit` |
| Bound total runtime | `timeout` |
| Cap network bandwidth | `trickle` / `wondershaper` |

Real-world example — the combo that fixed a Minecraft server watchdog
crash caused by a concurrent map-render batch job stealing CPU/disk I/O:
```bash
nice -n 19 ionice -c 3 python3 render.py -p 2 --config=render.conf
```
(`-p 2` there is the *app's own* flag capping its worker-process count —
worth checking if the tool has one, but `nice`/`ionice` work regardless of
whether it does.)
