# Networking Cheat Sheet

## Connectivity checks
```bash
ping -c 4 example.com                     # 4 pings then stop
ping -c 4 -i 0.2 example.com               # faster interval
traceroute example.com                       # path to host
mtr example.com                               # live traceroute + ping stats
nc -zv example.com 443                          # test if port is open (TCP)
nc -zvu example.com 53                            # test UDP port
telnet example.com 80                               # legacy manual port test
```

## DNS
```bash
dig example.com                            # full DNS query
dig +short example.com                      # just the IP
dig example.com MX                            # mail records
dig example.com NS                              # nameservers
dig @8.8.8.8 example.com                          # query specific DNS server
dig -x 8.8.8.8                                      # reverse lookup
nslookup example.com
host example.com
whois example.com
```

## Sockets & ports
```bash
ss -tuln                          # listening TCP/UDP sockets (modern netstat)
ss -tunap                          # + process info (needs sudo)
ss -s                                # socket summary stats
netstat -tulnp                        # legacy equivalent
lsof -i :8080                          # what's using port 8080
lsof -i -P -n | grep LISTEN              # all listening processes
fuser -k 8080/tcp                          # kill whatever holds port 8080
```

## Interfaces & routing
```bash
ip addr show                       # list interfaces + IPs (modern ifconfig)
ip a                                 # shorthand
ip link show                          # interface status
ip route show                           # routing table
ip route get 8.8.8.8                      # which route/interface for a destination
ifconfig                                    # legacy interface info
route -n                                      # legacy routing table
```

## Firewall (iptables / ufw)
```bash
sudo ufw status verbose
sudo ufw allow 22/tcp
sudo ufw allow from 10.0.0.0/24 to any port 5432
sudo ufw deny 8080
sudo iptables -L -n -v                    # list rules
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

## Bandwidth / throughput
```bash
iperf3 -s                          # run server side
iperf3 -c server_ip                  # run client side test
speedtest-cli                          # internet speed test
nload                                    # live bandwidth per interface
iftop                                      # live bandwidth per connection
```

## Packet capture
```bash
sudo tcpdump -i eth0                          # capture on interface
sudo tcpdump -i eth0 port 443
sudo tcpdump -i eth0 host 10.0.0.5
sudo tcpdump -i eth0 -w capture.pcap            # save to file
sudo tcpdump -r capture.pcap                      # read from file
sudo nmap -p 1-1000 example.com                     # port scan (authorized targets only)
sudo nmap -sV example.com                             # service/version detection
```

## Useful one-liners
```bash
curl -s ifconfig.me                                  # public IP
hostname -I                                            # local IPs
getent hosts example.com                                # resolve via NSS (like /etc/hosts + DNS)
ss -tn state established '( dport = :443 )'               # active HTTPS connections
watch -n1 'ss -s'                                            # live socket stats
python3 -m http.server 8000                                   # quick local file server
```
