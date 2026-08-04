# FreeBSD Cheat Sheet

## Package management (pkg)
```bash
pkg install nginx
pkg remove nginx
pkg update                      # refresh repo catalog
pkg upgrade                     # upgrade all installed packages
pkg search nginx
pkg info nginx                  # package details
pkg info                        # list installed packages
pkg which /usr/local/bin/nginx  # which package owns a file
pkg autoremove                  # remove orphaned dependencies
pkg clean                       # clear package cache
pkg audit -F                    # check installed packages for known vulns
pkg version -v                  # compare installed vs latest versions
```

## Ports tree (build from source)
```bash
portsnap fetch extract  # initial ports tree fetch
portsnap fetch update   # update existing ports tree
cd /usr/ports/www/nginx && make install clean
make config             # choose build options interactively
make search name=nginx  # search ports tree
portmaster -a           # upgrade all installed ports (if portmaster installed)
```

## Services (rc.d / rc.conf)
```bash
service nginx start
service nginx stop
service nginx restart
service nginx status
service -e                # list enabled services
sysrc nginx_enable="YES"  # enable at boot (writes to /etc/rc.conf)
sysrc -a                  # dump all rc.conf variables
cat /etc/rc.conf          # view boot-time service config
/etc/rc.d/nginx onestart  # start without enabling
```

## Users & permissions
```bash
adduser                          # interactive user creation
pw useradd chris -m -s /bin/csh  # scriptable user creation
pw groupmod wheel -m chris       # add user to wheel (sudo-equivalent) group
pw usermod chris -s /usr/local/bin/bash
rmuser chris
```

## ZFS (common on FreeBSD)
```bash
zpool status
zpool list
zfs list
zfs create pool/dataset
zfs snapshot pool/dataset@backup1
zfs list -t snapshot
zfs rollback pool/dataset@backup1
zfs send pool/dataset@backup1 | zfs receive backup_pool/dataset
zfs set compression=lz4 pool/dataset
zfs get all pool/dataset
```

## Jails (lightweight containers)
```bash
jls              # list running jails
jail -c path=/jails/myjail name=myjail ip4.addr=10.0.0.5 command=/bin/sh
jexec myjail sh  # shell into a jail
service jail start myjail
iocage list      # if using iocage jail manager
iocage create -n myjail -r 13.2-RELEASE
```

## System info
```bash
uname -a
freebsd-version
sysctl hw.model           # CPU model
sysctl -a | grep hw.ncpu  # CPU count
top
gstat                     # live disk I/O stats
dmesg | tail
```

## Useful one-liners
```bash
pkg install -y $(cat pkglist.txt)             # bulk install from a list
freebsd-update fetch install                  # patch base system
zfs list -o name,used,avail,refer,mountpoint  # readable ZFS overview
service -e | xargs -I{} service {} status     # status of all enabled services
find / -flags schg 2>/dev/null                # find immutable files (schg flag)
```
