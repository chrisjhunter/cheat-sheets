# Red Hat / RHEL / CentOS / Fedora Cheat Sheet

## Package management (dnf / yum)
```bash
dnf install nginx                     # install (RHEL8+/Fedora); use `yum` on RHEL7/CentOS7
dnf remove nginx
dnf update                              # update all packages
dnf update nginx                          # update one package
dnf search nginx                            # search repos
dnf info nginx                                # package details
dnf list installed                              # list installed packages
dnf list installed | grep nginx
dnf history                                       # transaction history
dnf history undo 12                                 # roll back a transaction
dnf provides */nginx                                  # find which package provides a file/binary
dnf repolist                                             # enabled repos
dnf clean all                                              # clear cache
dnf makecache
dnf -y install httpd                                          # auto-confirm
dnf group install "Development Tools"                            # install package group
```

## RPM (low-level)
```bash
rpm -qa                             # list all installed packages
rpm -qi nginx                         # package info
rpm -ql nginx                           # list files owned by package
rpm -qf /etc/nginx/nginx.conf             # which package owns this file
rpm -ivh package.rpm                        # install a local rpm
rpm -Uvh package.rpm                          # upgrade
rpm -e nginx                                    # remove
rpm --verify nginx                                # check for modified/missing files
```

## systemd (service management)
```bash
systemctl start nginx
systemctl enable --now nginx           # enable at boot + start now
systemctl status nginx
systemctl restart nginx
systemctl daemon-reload                  # after editing unit files
systemctl list-units --type=service --state=running
systemctl is-enabled nginx
systemctl mask nginx                       # prevent it from starting at all
```

## Firewalld
```bash
firewall-cmd --state
firewall-cmd --list-all                       # active zone rules
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload                            # apply --permanent changes
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --get-zones
firewall-cmd --zone=public --list-ports
```

## SELinux
```bash
getenforce                            # current mode: Enforcing/Permissive/Disabled
setenforce 0                            # temporarily set permissive
sestatus                                  # detailed status
ls -Z file                                  # view SELinux context of a file
chcon -t httpd_sys_content_t file             # change context (temporary)
restorecon -Rv /var/www                         # restore default contexts
semanage port -a -t http_port_t -p tcp 8080       # allow a service to use a nonstandard port
semanage fcontext -a -t httpd_sys_content_t "/srv/www(/.*)?"   # persistent context rule
audit2why < /var/log/audit/audit.log              # explain a denial
ausearch -m avc -ts recent                          # recent SELinux denials
```

## Networking (NetworkManager)
```bash
nmcli device status
nmcli connection show
nmcli connection up eth0
nmcli device wifi list
nmcli connection modify eth0 ipv4.addresses 10.0.0.5/24 ipv4.method manual
nmcli connection down eth0 && nmcli connection up eth0
```

## Useful one-liners
```bash
dnf history list | head -20                                # recent package changes
rpm -qa --last | head -20                                     # recently installed packages
journalctl -p err -b                                             # errors since last boot
subscription-manager status                                        # RHEL subscription status
ausearch -m avc -ts today | audit2allow -M mypolicy                  # generate SELinux policy from denials
firewall-cmd --get-active-zones
```
