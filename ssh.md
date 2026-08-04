# SSH Cheat Sheet

## Basics
```bash
ssh user@host                       # connect
ssh -p 2222 user@host               # custom port
ssh -i ~/.ssh/id_ed25519 user@host  # specific key
ssh -v user@host                    # verbose (debug connection issues), -vvv for more
```

## Key management
```bash
ssh-keygen -t ed25519 -C "email@example.com"      # generate modern keypair
ssh-keygen -t rsa -b 4096 -C "email@example.com"  # RSA fallback for old systems
ssh-copy-id user@host                             # push pubkey to remote authorized_keys
ssh-add ~/.ssh/id_ed25519                         # add key to agent
ssh-add -l                                        # list keys in agent
eval "$(ssh-agent -s)"                            # start agent
```

## Config file (~/.ssh/config)
```
Host myserver
    HostName 192.168.1.10
    User chris
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    ForwardAgent yes

Host *.internal
    ProxyJump bastion
```
Then just: `ssh myserver`

## Copying files
```bash
scp file.txt user@host:/path/            # copy file to remote
scp -r dir/ user@host:/path/             # copy dir recursively
scp user@host:/path/file.txt .           # copy from remote
rsync -avz -e ssh src/ user@host:/path/  # sync (better than scp for large/incremental)
sftp user@host                           # interactive file transfer
```

## Tunneling & forwarding
```bash
ssh -L 8080:localhost:80 user@host               # local port forward (local:remote)
ssh -R 9000:localhost:3000 user@host             # remote port forward
ssh -D 1080 user@host                            # dynamic SOCKS proxy
ssh -N -f -L 5432:db-internal:5432 user@bastion  # background tunnel, no shell
ssh -J bastion user@internal-host                # jump host (ProxyJump)
```

## Running remote commands
```bash
ssh user@host "ls -la /var/log"
ssh user@host 'bash -s' < local_script.sh      # run local script remotely
ssh user@host uptime && ssh user@host df -h    # chain commands
cat file.txt | ssh user@host "cat > file.txt"  # pipe content to remote file
```

## Sessions & keepalive
```bash
ssh -o ServerAliveInterval=60 user@host    # keep connection alive
tmux new -s work; ssh user@host            # persistent session survives disconnects
ssh -o ConnectTimeout=5 user@host echo ok  # fail fast if unreachable
```

## Troubleshooting
```bash
ssh -T git@github.com                      # test github auth
ssh-keygen -R hostname                     # remove stale host key (after IP reuse)
chmod 600 ~/.ssh/id_ed25519                # fix "permissions too open" error
chmod 700 ~/.ssh                           # correct dir perms
ssh -o StrictHostKeyChecking=no user@host  # skip host key check (CI use only)
```
