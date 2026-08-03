# illumos / Solaris Cheat Sheet

> Applies to illumos distributions (OmniOS, SmartOS, OpenIndiana) and Solaris/Oracle Solaris. Notably different from Linux: SMF instead of init/systemd, IPS instead of apt/yum, native ZFS, Zones instead of containers, DTrace.

## SMF (Service Management Facility) — replaces init.d/systemd
```bash
svcs                                # list all service instances
svcs -a                                # all services incl. disabled
svcs nginx                               # status of a specific service
svcs -x                                    # list services in maintenance/failure state
svcs -l svc:/network/nginx:default           # detailed info on a service
svcadm enable nginx
svcadm disable nginx
svcadm restart nginx
svcadm refresh nginx                           # reload config (like SIGHUP)
svcadm clear nginx                               # clear a service from maintenance state
svccfg -s nginx listprop                           # list service properties
svccfg -s nginx setprop config/port = 8080           # set a property
tail -f $(svcs -L nginx)                               # follow a service's log file
```

## IPS (Image Packaging System) — package management
```bash
pkg install nginx
pkg uninstall nginx
pkg update                          # update all packages
pkg refresh                           # refresh publisher catalogs
pkg search nginx
pkg info nginx
pkg list                                # installed packages
pkg list -a                               # all available packages
pkg contents nginx                          # files owned by a package
pkg publisher                                 # configured package repos
pkg history                                     # transaction history
pkg verify nginx                                  # verify package integrity
beadm list                                          # boot environments (safe upgrade rollback)
beadm create pre-upgrade-BE
beadm activate pre-upgrade-BE
```

## Zones (native virtualization, like FreeBSD jails)
```bash
zoneadm list -cv                    # list all zones + state
zonecfg -z myzone                     # configure a zone (interactive)
zoneadm -z myzone install
zoneadm -z myzone boot
zoneadm -z myzone halt
zlogin myzone                           # log into a running zone
zlogin myzone bash -c "uptime"            # run single command in a zone
zoneadm -z myzone uninstall
```

## ZFS (native, deeply integrated)
```bash
zpool status
zpool list
zpool history                       # commands that modified the pool
zfs list
zfs create pool/dataset
zfs snapshot pool/dataset@backup1
zfs rollback pool/dataset@backup1
zfs send pool/dataset@backup1 | zfs receive backup_pool/dataset
zfs set compression=lz4 pool/dataset
zfs get all pool/dataset
```

## DTrace (dynamic tracing)
```bash
dtrace -l | wc -l                              # count available probes
dtrace -n 'syscall::open:entry { printf("%s %s", execname, copyinstr(arg0)); }'
dtrace -n 'proc:::exec-success { printf("%s\n", curpsinfo->pr_psargs); }'   # trace new processes
dtruss cmd                                        # syscall trace for a command (like strace)
dtrace -n 'io:::start { @[execname] = count(); }'    # I/O ops per process, live
```

## System info & processes
```bash
uname -a
prtconf                            # hardware config
psrinfo -v                           # CPU info
prstat                                 # like top (per-process stats)
prstat -Z                                # per-zone resource usage
mpstat 1                                   # per-CPU stats
iostat -xn 1                                 # disk I/O stats
vmstat 1                                       # memory/swap stats
ps -ef
pfiles <pid>                                     # open files for a process (like lsof)
pstack <pid>                                       # stack trace of a running process
truss -p <pid>                                       # attach and trace syscalls live
```

## Networking
```bash
dladm show-link                     # list data links (interfaces)
dladm show-phys
ipadm show-addr                       # list IP addresses
ipadm create-addr -T static -a 10.0.0.5/24 net0/v4
route -p show                           # persistent routes
snoop -d net0                             # packet capture (illumos equivalent of tcpdump)
```

## Useful one-liners
```bash
svcs -xv                                                   # verbose explanation of failed services
pkg list -H | wc -l                                          # count installed packages
zfs list -t snapshot -o name,used,creation                     # snapshot disk usage
prstat -s cpu 1 5                                                 # top CPU consumers, 5 samples
dtrace -n 'BEGIN { trace("tracing..."); }' -n 'tick-5s { exit(0); }'  # quick 5s dtrace sanity check
beadm list -H                                                          # scriptable boot-env listing
```
