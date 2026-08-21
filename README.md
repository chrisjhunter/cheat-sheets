# Cheat Sheets

A personal collection of quick-reference cheat sheets for common CLI tools and infra. Optimized for one-liners and everyday usage, not exhaustive docs.

## Index

### CLI tools
- [Bash](bash.md)
- [Git](git.md) ([workflows](git-workflows.md))
- [grep](grep.md)
- [sed](sed.md)
- [awk](awk.md)
- [SSH](ssh.md)
- [tmux](tmux.md)
- [curl](curl.md)
- [Netcat (nc)](netcat.md)
- [jq](jq.md)
- [Vim](vim.md)
- [OpenSSL / TLS](openssl.md)
- [GNU parallel](parallel.md)
- [Taskwarrior & Timewarrior](taskwarrior.md)
- [Go](go.md)

### Infra & platforms
- [Docker](docker.md)
- [Kubernetes](kubernetes.md)
- [Helm](helm.md)
- [Nomad](nomad.md)
- [Consul](consul.md)
- [Vault](vault.md)
- [Terraform](terraform.md)
- [Nginx](nginx.md)
- [Networking](networking.md)
- [Databases (Postgres / MySQL)](databases.md)
- [AWS CLI](aws-cli.md)
- [B2 CLI](b2.md) — Backblaze B2 cloud storage
- [Prometheus / PromQL](prometheus.md)

### Sysadmin / OS-specific
- [General Sysadmin](sysadmin.md)
- [Resource Control](resource-control.md) — nice, ionice, cgroups/systemd-run, taskset, cpulimit, ulimit, timeout
- [Red Hat / RHEL / Fedora](redhat.md)
- [Debian / Ubuntu](debian.md)
- [FreeBSD](freebsd.md)
- [illumos / Solaris](illumos.md)

### Reference
- [Important Numbers](numbers.md) — powers of 2, CIDR blocks, nines of uptime, ports, HTTP/exit codes, and more
- [Temp Files & Scratch Space](tmpfiles.md) — mktemp, cleanup traps, atomic writes, and why

## Usage

Each file is self-contained Markdown, grep-friendly. Search across all sheets:

```bash
grep -ri "keyword" *.md
```
