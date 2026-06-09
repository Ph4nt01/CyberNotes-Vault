## Basic connectivity & reachability

| Tool                           |                                                     Purpose / When to use | Essential flags / options                                                                | Example(s)                    | Key output fields                                                           | Privileges | Pentest notes / pitfalls                                                        |
| ------------------------------ | ------------------------------------------------------------------------: | ---------------------------------------------------------------------------------------- | ----------------------------- | --------------------------------------------------------------------------- | ---------: | ------------------------------------------------------------------------------- |
| `ping`                         |                      ICMP echo to test reachability, latency, packet loss | `-c <count>` (Linux/Unix), `-i interval`, `-s payload`, `-W timeout`, `-t` (Windows TTL) | `ping -c 5 example.com`       | `time=` RTT ms; `% packet loss`; `ttl=` can hint at hops/OS                 |         no | Some hosts block ICMP or rate-limit ICMP; TTL reveals hop distance (approx).    |
| `traceroute` / `tracert` (Win) |                                      Discover path/hops by increasing TTL | `-I` use ICMP, `-T` use TCP, `-p` port, `-m max_ttl`                                     | `traceroute -T -p 443 target` | list of hops (IP/hostname), per-hop RTT samples; `*` means filtered/dropped |         no | Some networks filter UDP/ICMP so use TCP mode (`-T`) to test firewall behavior. |
| `mtr` (My Traceroute)          | Continuous traceroute + ping metrics, great for path/packet-loss analysis | `-r` report, `-w` wide, `-z` sort by loss                                                | `mtr --report example.com`    | per-hop loss %, latency min/avg/max                                         | usually no | Continuous view is useful for intermittent loss.                                |
| `pathping` (Windows)           |                       Windows combined traceroute + ping-style loss stats | n/a (interactive)                                                                        | `pathping example.com`        | per-hop latency & loss over time                                            |         no | Windows-only; slower but informative.                                           |

---
## Protocol/port testing & scanning

|Tool|Purpose|Essential flags|Example(s)|Interpret output|Privileges|Pentest notes|
|---|--:|---|---|---|--:|---|
|`nmap`|Port scanning, service/version detection, OS detection, scripts|`-sS` (SYN), `-sT` (connect), `-sU` (UDP), `-p` ports, `-A` (os+version+scripts), `-Pn` (no ping), `-T<0-5>` timing|`nmap -sS -p1-65535 -T4 -A target`|Open/closed/filtered; service names + versions; script output|no (privileges recommended for raw scans)|Many scan types (-sS requires raw sockets / root). Use `-Pn` to skip ICMP/ARP checks when hosts drop ICMP.|
|`masscan`|Very fast Internet-scale TCP port scanner (works by sending raw packets)|`-p` port(s), `--rate` packets/sec, `-oX` output|`masscan -p80,443 10.0.0.0/8 --rate=10000`|raw open port list|R|Careful — very noisy and easily blocked. Useful for wide reconnaissance.|
|`nping` (nmap project)|Probe networks with crafted packets (tcp/udp/icmp)|`--tcp`, `--udp`, `--icmp`, `--flags`, `--rate`|`nping --tcp -p 80 --flags S target`|Similar to hping: responses and timings|R for raw|Good for scripted packet flows and performance testing.|
|`hping3`|Craft TCP/UDP/ICMP packets, set flags, TTL, fragmentation|`-S/-A/-F/-R/-P` flags, `-p` port, `-c` count, `--flood`, `-s` src port|`hping3 -S -p 80 -c 3 target`|Shows replies, seq/ack, TTL|R|Useful for firewall evasion tests, SYN floods, TTL-based path tests, TCP fingerprinting.|
|`masscan`/`zmap`|Internet-scale scanning|n/a|n/a|n/a|R|Use responsibly — follow scope and law.|

---
## Connection/debugging and raw sockets

|Tool|Purpose|Key options|Example|Output interpretation|Privileges|
|---|--:|---|---|---|--:|
|`nc` / `netcat`|Read/write TCP/UDP sockets; banner grabbing; simple proxy|`-l` listen, `-p` port, `-u` UDP, `-v` verbose, `-k` keepalive|`nc -l -p 8080` or `nc target 80`|Writes/reads raw bytes. Useful for banner grabbing.|no (listen on low ports may need root)|
|`socat`|Like `netcat` but more features (UNIX sockets, TLS, proxying)|many; can do `TCP-LISTEN:443,reuseaddr,fork`|`socat TCP-LISTEN:8080,fork TCP:10.0.0.5:80`|proxying data / transforms|no|
|`telnet`|Connect to TCP port, manual protocol testing|`telnet host port`|`telnet example.com 25`|Connects and allows typing raw protocol lines|no|

---
## Packet capture & inspection

|Tool|Purpose|Key flags|Example|Important differences / reading tips|Privileges|
|---|--:|---|---|---|--:|
|`tcpdump`|Capture packets using libpcap. Lightweight CLI|`-i <iface>`, `-w file.pcap`, `-s snaplen`, `-nn` (no name lookup), `-A` (ascii), BPF filter e.g., `tcp port 80`|`sudo tcpdump -i eth0 -w capture.pcap tcp port 80`|`tcpdump` uses BPF filters (same as Wireshark capture filter). Output: timestamp, src:port > dst:port, flags [S.] seq ack win length|R|
|`tshark`|CLI version of Wireshark; powerful dissectors, display filters|`-i`, `-w`, `-Y` (display filter), `-f` (capture filter)|`sudo tshark -i eth0 -Y "http.request"`|`-f` uses BPF (capture filter). `-Y` uses Wireshark display filter syntax (different).|R|
|`wireshark`|GUI packet capture & deep analysis, protocol dissectors|filters, coloring rules, follow TCP stream|Open `capture.pcap` and use display filters `http` `tcp.flags.syn==1`|Capture filters (BPF) limit capture; display filters filter shown packets only. Learn disectors and TCP stream reassembly.|R|
|`ngrep`|grep-like for packet payloads (fast, ascii)|`-d` iface, `-W byline`, BPF filter|`sudo ngrep -d eth0 'password'`|Shows payload matching regex/strings|R|
|`pcap`/`pyshark` libs|Programmatic capture and parsing|n/a|`scapy`/`pyshark` scripts|Useful for automation|R for sniffing raw|
> **How to read a typical `tcpdump` line**
```
12:01:01.123456 IP 192.168.1.10.54321 > 93.184.216.34.80: Flags [S], seq 12345, win 64240, options [mss 1460,sackOK,TS val 123 ecr 0,nop,wscale 7], length 0
```
> - `12:01:...` timestamp
> - `IP src.port > dst.port`
> - `Flags [S]` (SYN) — TCP flags
> - `seq` and `ack` numbers — TCP sequence/ack values
> - `win` — window size (recv window)
> - `options` — TCP options like MSS, SACK, Timestamps
> - `length` — payload bytes in TCP segment

---
## Packet crafting & replay

| Tool               |                                                Purpose | Key options                                 | Example use                                       |        Privileges | Pentester use-cases                                                                  |
| ------------------ | -----------------------------------------------------: | ------------------------------------------- | ------------------------------------------------- | ----------------: | ------------------------------------------------------------------------------------ |
| `scapy` (Python)   | Programmatic packet crafting & sniffing; very flexible | Python API, send(), sr(), sniff(), rdpcap() | `send(IP(dst="1.2.3.4")/TCP(dport=80,flags="S"))` | R for raw sockets | Create custom packets (SYN floods, crafted HTTP, TTL hops), fuzzing, protocol tests. |
| `tcpreplay`        |                    Re-inject pcap traffic onto network | `--mbps`, `--loop`, `--preload-pcap`        | `tcpreplay --intf1=eth0 capture.pcap`             |                 R | Replay capture for IDS evasion tests or lab simulation.                              |
| `hping3`           |                                            (see above) | flags, spoofing `-a`, TTL `-t`              | `hping3 -S -p 80 -s 1234 -a 10.0.0.5 target`      |                 R | IP spoofing, TTL evasion, firewall tests.                                            |
| `raw sockets` libs |                           Low-level socket programming | language-specific                           | C/Python raw socket                               |                 R | Build custom tools.                                                                  |

---
## Service & application-layer testing (HTTP / TLS / DNS)

|Tool|Purpose|Key flags|Example|Privileges|Notes|
|---|--:|---|---|--:|---|
|`curl`|HTTP client/tooling (requests, TLS options)|`-I` headers only, `-k` ignore certs, `-v` verbose, `--resolve`, `--proxy`|`curl -v -k https://target`|no|Use `--cert` for client certs; `--resolve` to override DNS.|
|`httpie`|Human-friendly HTTP client|JSON friendly|`http GET example.com`|no|Better UX for interactive use.|
|`openssl s_client`|Test TLS handshakes, cert details, ALPN|`-connect host:443 -servername host -showcerts`|`openssl s_client -connect example.com:443 -servername example.com`|no|Inspect certificates, cipher negotiated, session ticket.|
|`ss` / `netstat`|List sockets and listening ports|`ss -tulnp`, `ss -s`|`ss -tulnp`|no (privileged for some)|`ss` is modern replacement for `netstat`. See PID/Proc using sockets.|
|`dig` / `nslookup` / `host`|DNS queries and debugging|`dig +short`, `dig @server`|`dig A example.com`|no|Use `+trace` / `@` server to check delegation.|
|`dnsenum` / `dnsrecon`|DNS enumeration for recon|n/a|n/a|no|Useful for pentesting subdomain discovery.|

---
## Routing, IP addressing, ARP, neighbor discovery

|Tool|Purpose|Key options|Example|Output interpretation|Privileges|
|---|--:|---|---|---|--:|
|`ip` (iproute2)|Configure/show IP addresses, routes, links|`ip addr`, `ip link set`, `ip route`|`ip addr show`, `ip route show`, `ip link set dev eth0 up`|`ip addr` shows IPs, scope; `ip route` shows routing table (default, network routes)|R for changing|
|`route` (legacy)|Show routing table (older)|n/a|`route -n`|legacy|no|
|`arp` / `arping` / `ip neigh`|ARP cache and testing layer 2 reachability|`arp -a`, `arping -c 3 192.168.1.1`|`arp -a` lists MACs; `arping` sends ARP requests|ARP cache entries with MAC and interface|arp cache viewing no, arping R|
|`ndisc6` / `ip -6 neigh`|IPv6 neighbor discovery|n/a|`ip -6 neigh show`|shows IPv6 neighbours|R for injections|
|`ethtool`|NIC capabilities and stats|`ethtool eth0`, `-S` stats|`ethtool -S eth0`|shows speed, duplex, errors, offload features|R for changes|

---
## Firewall, NAT, packet filtering and shaping

| Tool                   |                                           Purpose | Key options                                            | Example                                   | Notes                                              | Privileges |
| ---------------------- | ------------------------------------------------: | ------------------------------------------------------ | ----------------------------------------- | -------------------------------------------------- | ---------: |
| `iptables` / `nft`     |                   Packet filtering / NAT on Linux | chains, tables, rules syntax                           | `iptables -L -n -v` or `nft list ruleset` | Shows firewall rules; `iptables -t nat -L` for NAT |          R |
| `pf` (BSD)             | Packet filter on BSD/macOS (modern macOS uses pf) | `pfctl -sr` settings                                   | `sudo pfctl -sr`                          | macOS / OpenBSD environment specific               |          R |
| `tc` (traffic control) |              Shaping, qdisc, netem for impairment | `tc qdisc add dev eth0 root netem delay 100ms loss 1%` | `tc qdisc show`                           | Simulate latency/loss for testing                  |          R |

---
## Throughput, load and latency tools

|Tool|Purpose|Key options|Example|Notes|Privileges|
|---|--:|---|---|---|--:|
|`iperf3`|Bandwidth/throughput test (TCP/UDP)|`-c` client, `-s` server, `-u` UDP, `-P` parallel|`iperf3 -s` (server) and `iperf3 -c host -P 4`|Measures throughput, jitter (UDP)|usually no|
|`nuttcp`|Similar to iperf with different metrics|n/a|`nuttcp -S`|alternative|usually no|
|`ping` with large payloads|Check fragmentation/path-MTU|`ping -M do -s <size>`|`ping -M do -s 1472 target`|If fail, path MTU lower than attempted size|no|

---
## System socket / connection viewers

| Tool      |                            Purpose | Key flags / usage                            | Example             | Notes                                    |                     Privileges |
| --------- | ---------------------------------: | -------------------------------------------- | ------------------- | ---------------------------------------- | -----------------------------: |
| `ss`      | Show established/listening sockets | `ss -tulnp` list TCP/UDP listening with PIDs | `ss -tulnp          | grep :80`                                | Shows PID/process using socket |
| `lsof -i` |       List open sockets by process | `lsof -iTCP -sTCP:LISTEN`                    | `sudo lsof -i :443` | Useful to find process binding to a port |             R for some outputs |

---
## Windows-specific

| Tool                            |                      Purpose / Win equivalent | Example                                                  | Notes                                          |
| ------------------------------- | --------------------------------------------: | -------------------------------------------------------- | ---------------------------------------------- |
| `netsh` (Windows)               |    Network configuration, firewall, interface | `netsh interface ipv4 show config`                       | Powerful Windows CLI for many networking tasks |
| PowerShell `Test-NetConnection` |                      Ping/traceroute/tcp test | `Test-NetConnection -ComputerName example.com -Port 443` | Good single-command check                      |
| `Get-NetTCPConnection`          | Inspect TCP connections in Windows PowerShell | `Get-NetTCPConnection -State Established`                | Equivalent to `ss`/`netstat`                   |

---
## DNS tools

| Tool                   |                                Purpose | Key options                           | Example                               | Notes                                    |
| ---------------------- | -------------------------------------: | ------------------------------------- | ------------------------------------- | ---------------------------------------- |
| `dig`                  |               DNS querying & debugging | `@server`, `+short`, `+trace`, `AXFR` | `dig @8.8.8.8 example.com ANY +trace` | Use `+short` for script-friendly outputs |
| `dnsenum` / `dnsrecon` | Enumeration and zone transfer attempts | n/a                                   | `dnsrecon -d domain.com`              | Pentesting recon staples                 |
| `dnswalk`              |                Zone consistency checks | n/a                                   | n/a                                   | Specialized                              |

---
## TLS / Cert / HTTP specifics

|Tool|Purpose|Useful options|Example|What to look for|
|---|--:|---|---|---|
|`openssl s_client`|Test TLS handshake/certs|`-connect`, `-servername`, `-showcerts`, `-ALPN`|`openssl s_client -connect target:443 -servername example.com`|Check cert chain, negotiated cipher, session reuse, TLS version|
|`sslyze`|TLS scanner (ciphers, protocols, certs)|options to check certs/ciphers/SNI|`sslyze --regular target:443`|Automated TLS security posture|
|`curl -v --http2`|HTTP/2 negotiation, headers, cookie behavior|`--resolve`, `-I`|`curl -vkI https://example.com`|Check redirects, HSTS, cookies, TLS renegotiation|

---
## Monitoring / long-term observation

|Tool|Purpose|Example|Notes|
|---|--:|---|---|
|`iftop`|Live per-connection bandwidth usage|`sudo iftop -i eth0`|Interactive top-like for bandwidth|
|`nethogs`|Per-process network bandwidth|`sudo nethogs`|Useful to detect which process is using traffic|
|Prometheus + Grafana|Metrics & dashboards|exporters for network metrics|long-term monitoring|

---
## Fundamentals you should always remember
> - **3-way handshake**: SYN -> SYN/ACK -> ACK (connection established)
> - **TCP flags**: SYN, ACK, FIN, RST, PSH, URG - used for connection lifecycle & hijacking detection
> - **Seq / Ack**: sequence numbers + ack numbers important for stream reassembly & injection attacks
> - **Window size & scaling**: flow control - can lead to performance tuning/DoS vectors
> - **MTU & fragmentation**: Path MTU influences fragmentation; `ping -M do -s` helps test PMTUD
> - **Keepalives/timeouts**: affect long-lived connections; can influence session persistence tests
> - **NAT/SNAT/DNAT**: source/destination address translation changes source/destination IP seen by server/client
> - **ICMP types**: Destination Unreachable (codes matter), Time Exceeded (used by traceroute), Redirect — useful signals.
> - **ARP vs ND**: ARP IPv4, Neighbor Discovery (ND) for IPv6 — both are layer 2 discovery/poisoning targets in attacks.
## Common pitfalls / troubleshooting tips
> - **Offloading on NICs (TSO/GSO/LRO) distorts captures.** Disable offloads if packet capture doesn't look right (`ethtool -K eth0 gro off gso off tso off`).
> - **Asymmetric routing**: capture on one host may not show both directions - use SPAN/mirror or capture on both ends.
> - **Time sync**: packet timestamps need NTP for multi-host correlation.
> - **Root required** for raw sockets / capture / packet injection — on Linux capabilities can be set for single binaries (`setcap cap_net_raw+ep /usr/sbin/...`), but many tools still expect `sudo`.
> - **ICMP/trace limitations**: Hosts that filter ICMP give `* * *` hops - try TCP traceroute.
> - **HTTP/2 & TLS multiplexing**: tcpdump may show single TLS stream; use Wireshark HTTP/2 dissector with TLS keys (SSLKEYLOGFILE) to decrypt if you have keys.
> - **Proxy/transparent interception**: `curl --proxy` and browser config can change where traffic goes; a local proxy (Burp) won't show enterprise proxy behavior.
> - **Legal & scope**: scanning/crafting must be permitted by engagement rules - mass scanning and traffic replay are very noisy.

---