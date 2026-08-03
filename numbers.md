# Numbers Every Sysadmin/SRE Should Know

## Powers of 2
```
2^1   = 2
2^2   = 4
2^3   = 8
2^4   = 16
2^5   = 32
2^6   = 64
2^7   = 128
2^8   = 256
2^9   = 512
2^10  = 1,024              (1 Ki)
2^16  = 65,536              (max port number + 1; uint16 range)
2^20  = 1,048,576            (1 Mi)
2^24  = 16,777,216            (256^3; /8 network size; RGB color space)
2^30  = 1,073,741,824          (1 Gi)
2^31  = 2,147,483,648            (max signed int32 + 1; Unix Y2038 boundary)
2^32  = 4,294,967,296              (max uint32 + 1; total IPv4 address space)
2^40  = 1,099,511,627,776            (1 Ti)
2^63  = 9,223,372,036,854,775,808      (max signed int64)
2^64  = 18,446,744,073,709,551,616       (max uint64)
```

## Binary vs decimal prefixes (the 1000 vs 1024 trap)
```
1 KB = 1,000 bytes     1 KiB = 1,024 bytes
1 MB = 1,000,000       1 MiB = 1,048,576
1 GB = 1,000,000,000   1 GiB = 1,073,741,824
1 TB = 10^12           1 TiB = 2^40 (1,099,511,627,776)
```
Disk vendors market in decimal (GB); OSes usually report binary (GiB, but often label it "GB") — this is why a "1TB" drive shows ~931 GB free.

## CIDR / subnet cheat table (IPv4)
```
/32   1            single host
/31   2            point-to-point link (RFC 3021)
/30   4  (2 usable)  smallest usable LAN subnet
/29   8  (6 usable)
/28   16 (14 usable)
/27   32 (30 usable)
/26   64 (62 usable)
/25   128 (126 usable)
/24   256 (254 usable)  "a /24", classic Class C, 255.255.255.0
/23   512
/22   1,024
/21   2,048
/20   4,096
/16   65,536            255.255.0.0, classic Class B
/13   524,288            common cloud VPC size
/8    16,777,216          255.0.0.0, classic Class A
/0    4,294,967,296        entire IPv4 space
```
Subnet mask quick reference:
```
/24 = 255.255.255.0     /27 = 255.255.255.224
/25 = 255.255.255.128   /28 = 255.255.255.240
/26 = 255.255.255.192   /29 = 255.255.255.248
                          /30 = 255.255.255.252
```

## IPv6 sizing
```
/128   single host
/64    standard LAN subnet (18,446,744,073,709,551,616 addresses)
/56    common ISP delegation to a site
/48    common ISP delegation to a customer/business (65,536 /64s)
/32    typical allocation to an ISP
```

## Reserved / private IPv4 ranges
```
10.0.0.0/8         10.0.0.0    - 10.255.255.255   (16.7M addresses)
172.16.0.0/12      172.16.0.0  - 172.31.255.255    (1.05M addresses)
192.168.0.0/16     192.168.0.0 - 192.168.255.255     (65,536 addresses)
127.0.0.0/8        loopback
169.254.0.0/16     link-local
100.64.0.0/10      carrier-grade NAT (CGNAT)
```

## Nines of availability (uptime)
```
90%      ("one nine")     36.5 days/year downtime
99%      ("two nines")    3.65 days/year   |  87.6 hrs/year
99.9%    ("three nines")  8.76 hrs/year    |  43.8 min/month
99.95%                    4.38 hrs/year    |  21.9 min/month
99.99%   ("four nines")   52.6 min/year    |  4.38 min/month
99.999%  ("five nines")   5.26 min/year    |  25.9 sec/month
99.9999% ("six nines")    31.5 sec/year
```

## Common port numbers
```
20/21      FTP (data/control)
22         SSH
23         Telnet
25         SMTP
53         DNS
67/68      DHCP
69         TFTP
80         HTTP
110        POP3
111        RPC
123        NTP
143        IMAP
161/162    SNMP
179        BGP
389        LDAP
443        HTTPS
445        SMB
465        SMTPS
514        Syslog
587        SMTP submission
631        IPP/CUPS
636        LDAPS
993        IMAPS
995        POP3S
1433       MSSQL
1521       Oracle DB
2049       NFS
2181       ZooKeeper
2379/2380  etcd
3000       common dev server
3306       MySQL
3389       RDP
5432       PostgreSQL
5601       Kibana
5672       RabbitMQ (AMQP)
5900       VNC
6379       Redis
6443       Kubernetes API server
8080       common HTTP alt
8200       Vault
8300-8302  Consul (server RPC/serf)
8500       Consul HTTP API
8600       Consul DNS
9000       common app/Portainer/MinIO
9042       Cassandra
9090       Prometheus
9092       Kafka
9200/9300  Elasticsearch
27017      MongoDB
```
Well-known ports: 0-1023 (require root/CAP_NET_BIND_SERVICE). Registered: 1024-49151. Dynamic/ephemeral: 49152-65535.

## HTTP status codes (the ones you actually see)
```
200 OK                  400 Bad Request           500 Internal Server Error
201 Created             401 Unauthorized          501 Not Implemented
204 No Content          403 Forbidden             502 Bad Gateway
301 Moved Permanently   404 Not Found             503 Service Unavailable
302 Found (temp redir)  405 Method Not Allowed    504 Gateway Timeout
304 Not Modified        408 Request Timeout       507 Insufficient Storage
307 Temporary Redirect  409 Conflict
308 Permanent Redirect  410 Gone
                        413 Payload Too Large
                        422 Unprocessable Entity
                        429 Too Many Requests
```

## Exit codes (Bash / Linux)
```
0     success
1     general error
2     misuse of shell builtin
126   command found but not executable
127   command not found
128   invalid argument to exit
128+N killed by signal N (e.g. 130 = 128+2 = SIGINT/Ctrl-C, 137 = 128+9 = SIGKILL, 143 = 128+15 = SIGTERM)
```

## Common signals
```
1  SIGHUP    hangup / reload config
2  SIGINT    interrupt (Ctrl-C)
3  SIGQUIT   quit + core dump
9  SIGKILL   force kill, cannot be caught/ignored
15 SIGTERM   graceful terminate (default for `kill`)
18 SIGCONT   resume after SIGSTOP
19 SIGSTOP   pause process, cannot be caught
20 SIGTSTP   terminal stop (Ctrl-Z)
```

## File permission numbers (octal)
```
0 --- none        4 r--        6 rw-
1 --x execute      5 r-x        7 rwx
2 -w- write
3 -wx
common combos: 644 (rw-r--r--, typical file), 755 (rwxr-xr-x, typical dir/executable),
600 (rw-------, private key/secret), 700 (rwx------, private dir),
777 (rwxrwxrwx, avoid — world-writable), 4755 (setuid), 2755 (setgid), 1777 (sticky, e.g. /tmp)
```

## Time
```
60          seconds in a minute
3,600       seconds in an hour
86,400      seconds in a day
604,800     seconds in a week
2,592,000   seconds in 30 days
31,536,000  seconds in a non-leap year
1,000,000,000 (1e9)  seconds ≈ 31.7 years
1,700,000,000-ish    approx current Unix epoch time (2023-2024 era; grows ~31.5M/year)
2,147,483,647         max signed int32 Unix time -> overflow 2038-01-19 ("Year 2038 problem")
```

## Load average / CPU rule of thumb
```
load average == number of CPU cores  -> fully utilized, roughly saturated
load average <  number of cores       -> headroom
load average >  number of cores         -> processes queued/waiting (on Linux; includes I/O wait)
```

## Misc important numbers
```
1,500          default Ethernet MTU (bytes)
9,000          common "jumbo frame" MTU
65,535         max TCP/UDP port number (2^16 - 1); also common max value for uint16 fields
4,096          common disk block size (bytes); also common RSA key size (bits)
512            legacy disk sector size (bytes)
128            default TCP listen() backlog on many systems (historical default)
32,768         default Linux max PID (cat /proc/sys/kernel/pid_max, historically)
255            max Linux filename length (bytes, ext4/most filesystems)
4,096          max Linux path length (PATH_MAX, bytes)
1024           default `ulimit -n` (open file descriptors) on many distros
0.0.0.0/0      "all IPv4 addresses" (default route / open to the world in a SG rule)
::/0           "all IPv6 addresses" equivalent
```
