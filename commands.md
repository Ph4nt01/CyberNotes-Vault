### File Operations
```
file                                               #Inspect file type
find -size 1033c -name example.md                  #Find files by size/name
sort file.txt | uniq                               #Filter unique lines
xxd -r                                             #Revert hexdump to binary
diff                                               #Compare files/dirs
```

### Network Operations
```
scp -P 2220 user@host:file.txt ~/                  #Secure file copy
openssl s_client -connect host:port                #SSL/TLS inspection
ncat --ssl host port                               #SSL/TLS connection
nmap -sV host                                      #Service/port scanning
```

### Process Management
```
crontab -l                                         #View scheduled tasks
ps aux | grep root                                 #Find processes
kill -9 PID                                        #Force-kill process
```

### Privilege Escalation
```
sudo -l                                            #Check usable sudo rights
find / -perm -4000 2>/dev/null                     #Find SUID binaries
linpeas.sh                                         #Auto PEAS scanner
```

### Web Testing
```
sqlmap -u "http://site.com?param=1"                #SQL injection testing
ffuf -w wordlist -u http://site.com/FUZZ           #Directory fuzzing
curl -H "X-Header: test" http://site.com           #Custom header requests
```

### Forensics
```
strings file.bin | grep "flag{"                    #Extract embedded strings
exiftool image.jpg                                 #View metadata
binwalk -e firmware.bin                            #Extract embedded files
```

