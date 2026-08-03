# Debian / Ubuntu Cheat Sheet

## APT package management
```bash
apt update                            # refresh package index
apt upgrade                             # upgrade all packages
apt full-upgrade                          # upgrade + handle dependency changes/removals
apt install nginx
apt remove nginx                            # remove package, keep config
apt purge nginx                               # remove package + config files
apt autoremove                                  # remove unused dependencies
apt search nginx
apt show nginx                                    # package details
apt list --installed
apt list --installed | grep nginx
apt-cache policy nginx                              # installed vs available versions
apt-get source nginx                                  # download source package
apt edit-sources                                        # edit /etc/apt/sources.list
apt clean                                                 # clear downloaded .deb cache
```

## dpkg (low-level)
```bash
dpkg -l                             # list all installed packages
dpkg -l | grep nginx
dpkg -L nginx                         # list files owned by package
dpkg -S /etc/nginx/nginx.conf           # which package owns this file
dpkg -i package.deb                       # install a local .deb
dpkg -r nginx                               # remove
dpkg --configure -a                           # fix interrupted install
dpkg-reconfigure tzdata                         # rerun a package's config prompts
```

## systemd (service management)
```bash
systemctl start nginx
systemctl enable --now nginx
systemctl status nginx
systemctl restart nginx
systemctl daemon-reload
journalctl -u nginx -f
```

## UFW firewall
```bash
ufw status verbose
ufw enable
ufw allow 22/tcp
ufw allow from 10.0.0.0/24 to any port 5432
ufw deny 8080
ufw delete allow 8080
ufw app list                          # preconfigured app profiles
```

## Alternatives & configs
```bash
update-alternatives --list editor
update-alternatives --config editor       # choose default editor/binary interactively
debconf-show nginx                          # view debconf answers for a package
dpkg-reconfigure -plow package                # reconfigure with low priority prompts
```

## PPAs (Ubuntu)
```bash
add-apt-repository ppa:someuser/ppa
apt update
add-apt-repository --remove ppa:someuser/ppa
```

## Useful one-liners
```bash
apt list --upgradable                                     # what's pending an upgrade
grep ^ /etc/apt/sources.list /etc/apt/sources.list.d/*      # dump all configured repos
apt-mark hold nginx                                            # pin package, exclude from upgrades
apt-mark unhold nginx
apt-get install -f                                                # fix broken dependencies
dpkg --get-selections | grep -v deinstall                          # list all "installed" packages
journalctl -p err -b                                                  # errors since last boot
history | grep "apt install"                                            # recall what you've installed
```
