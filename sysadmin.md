# General Sysadmin Cheat Sheet

> Cross-distro basics. See [redhat.md](redhat.md), [debian.md](debian.md), [freebsd.md](freebsd.md), and [illumos.md](illumos.md) for OS-specific package/service managers.

## Users & groups
```bash
useradd -m -s /bin/bash chris          # create user with home dir + shell
usermod -aG sudo chris                  # add to group (append, don't overwrite)
userdel -r chris                          # delete user + home dir
passwd chris                                # set/change password
chage -l chris                                # view password expiry policy
groupadd developers
groups chris                                    # list a user's groups
id chris                                          # uid/gid/groups
su - chris                                          # switch user, load their env
sudo -u chris cmd                                     # run single command as another user
sudo -l                                                 # list what current user can sudo
who                                                       # who's logged in
w                                                           # who's logged in + what they're doing
last                                                          # login history
```

## Permissions & ownership
```bash
chmod 755 file                          # rwxr-xr-x
chmod u+x file                            # add execute for owner
chmod -R 644 dir/                           # recursive
chown user:group file
chown -R user:group dir/
umask                                           # current default permission mask
umask 022                                         # set default mask
chattr +i file                                      # make file immutable (even root can't rm)
lsattr file
setfacl -m u:chris:rwx file                           # extended ACLs
getfacl file
find / -perm -4000 -type f 2>/dev/null                  # find SUID binaries
```

## Processes
```bash
ps aux                                # all processes, BSD style
ps -ef                                  # all processes, SysV style
ps aux --sort=-%cpu | head              # top CPU consumers
top / htop                                # interactive process viewers
pgrep -f nginx                              # find PIDs by name/pattern
pkill -f nginx                                # kill by name/pattern
kill -15 1234                                   # graceful terminate (SIGTERM)
kill -9 1234                                      # force kill (SIGKILL)
nice -n 10 cmd                                      # run with lower priority
renice -n 5 -p 1234                                   # change priority of running process
nohup cmd &                                             # survive terminal close
disown -h %1
strace -p 1234                                            # trace syscalls of running process
lsof -p 1234                                                # open files for a PID
```

## System info
```bash
uname -a                          # kernel/arch info
hostnamectl                         # hostname + OS info (systemd)
uptime                                # load average + uptime
free -h                                 # memory usage
df -h                                     # disk usage
du -sh * | sort -h                          # dir sizes sorted
lscpu                                         # CPU details
lsblk                                           # block devices
lsusb / lspci                                     # USB/PCI devices
dmesg | tail -50                                    # recent kernel messages
cat /etc/os-release                                   # OS identification
```

## Logs
```bash
journalctl -xe                        # recent systemd journal, extra detail
journalctl -u nginx -f                  # follow logs for a unit
journalctl -u nginx --since "1 hour ago"
journalctl -k                             # kernel messages only
journalctl --disk-usage
journalctl --vacuum-time=7d                 # trim journal to last 7 days
tail -f /var/log/syslog                       # (Debian) or /var/log/messages (RHEL)
grep -i error /var/log/syslog
```

## Cron & scheduled tasks
```bash
crontab -e                        # edit current user's crontab
crontab -l                          # list current user's crontab
crontab -u chris -l                   # list another user's crontab
# m h dom mon dow command
0 2 * * * /usr/local/bin/backup.sh       # daily at 2am
*/15 * * * * /usr/local/bin/check.sh       # every 15 minutes
systemctl list-timers                        # systemd timers (modern cron alternative)
```

## Disks & filesystems
```bash
mount /dev/sdb1 /mnt/data
umount /mnt/data
lsblk -f                                # devices with filesystem info
blkid                                     # UUIDs of devices
fdisk -l                                    # partition tables
mkfs.ext4 /dev/sdb1                           # format partition
fsck /dev/sdb1                                  # check filesystem
mount -o remount,rw /                             # remount root read-write
cat /etc/fstab                                      # persistent mount config
df -i                                                 # inode usage
```

## Networking basics
```bash
hostname -I                       # local IPs
ip addr show
systemctl status networking          # or NetworkManager, depends on distro
```
See [networking.md](networking.md) for the full networking reference.

## Package/service management
See distro-specific sheets: [redhat.md](redhat.md) (yum/dnf, firewalld, SELinux), [debian.md](debian.md) (apt/dpkg), [freebsd.md](freebsd.md) (pkg/ports), [illumos.md](illumos.md) (SMF/IPS/zones).

## Useful one-liners
```bash
find / -xdev -type f -size +100M 2>/dev/null | sort           # big files, current filesystem only
find / -mtime -1 -type f 2>/dev/null                             # files modified in last day
du -h --max-depth=1 / 2>/dev/null | sort -h                        # disk hogs by top-level dir
watch -n1 'ps aux --sort=-%mem | head -10'                           # live top-memory processes
ss -tulnp                                                               # listening ports + processes
timedatectl                                                               # time/timezone info
timedatectl set-timezone America/New_York
history | awk '{print $2}' | sort | uniq -c | sort -rn | head          # most-used commands
```
