# Network discovery
## Identifying hosts
What do you need to scan, which subnets? While some are easy to find (just logic!), you may
need to investigate and find new subnets.

### Easy subnets
1. You can scan your own subnet:
```
$ ip addr
> ipconfig
```

2. Check your routes:
```
$ ip route
> route PRINT
```

3. Check your DNS configuration:
```
$ cat /etc/resolv.conf
> nslookup <dns_suffix>
```
> Where <dns_suffix> is the ipconfig's value of "Connection-specific DNS Suffix" field


4. Check your ARP table:
```
$ ip neigh
> arp /a
```

5. Investigate communication (sockets)
```
$ ss -t -a
$ netstat -ant
> netstat -a
```

6. Check if "use automatic configuration script" is checked within your Internet Explorer settings:
*Internet Options / Connections / LAN Settings*
If yes, copy the Address of the proxy.pac file and download it. You will get many subnets there.

### PowerView
Using [PowerView](tools/PowerView.md) to enumerate computers with *Get-DomainComputer*:
```
Get-DomainComputer
Get-DomainComputer -Domain <domain>
Get-DomainComputer -LDAPFilter <filter>
Get-DomainComputer -SearchBase <base>  
```
> Filtering with LDAPFilters or SearchBase is really interesting to exclude workstations.
For this, you can see the section [AD discovery](guidelines/internal/ADDiscovery.md)

### Guessing
If you're stuck with a poor /24, you may also try https://github.com/bik3te/Scripts/blob/master/range_guesser.py.
> It will generate the list of every first and last host's IP address of any
private range possible based on a specific netmask (/24 by default).
You can then use this list for scanning and assume the range is used if a host was found.

## Scanning
You finally know what to scan. First perform a quick scan in order to gain time and identify
interesting TCP services such as:

PORT  | Protocol
----- | --------
21    | FTP
22    | SSH
23    | Telnet
80    | HTTP
135	  | MS-RPC
139	  | Netbios
389   | LDAP
443   | HTTPS
445	  | SMB
512   | rexec
513   | rlogin
514   | rsh
1099  | RMI
1433	| MS-SQL
1521	| Oracle
2049  | NFS
3306	| MySQL
3389  | RDP
8080  | HTTP
8443  | HTTPS
11211	| Memcached

Without forgetting the UDP port 161 (SNMP)!

### Quick scan for sensitive services only
```
nmap -vvv -Pn -n -sS -A -sC -p U:161,T:21,22,23,25,80,135,139,389,443,445,512,513,514,1099,1433,1521,2049,3306,3389,8080,8443,11211 -T3 -oA 192.168.0_Quick 192.168.0.*
```

### Full scan
```
nmap -vvv -Pn -n -sS -A -sC -p- -T3 -oA 192.168.0_Full_TCP 192.168.0.*
```

### Nmap tips
1. Use intelligent names when exporting results:
* Specify the protocol
* Specify the subnet
* Specify the range
Ex: 192.168.0_Full_TCP.xml

2. For readability use a stylesheet to generate an HTML format of the report:
```
xsltproc 192.168.0_Full_TCP.xml -o 192.168.0_Full_TCP.html
```
Better again, download https://github.com/honze-net/nmap-bootstrap-xsl/blob/master/nmap-bootstrap.xsl
and generate the HTML report with bootstrap ;)
```
xsltproc 192.168.0_Full_TCP.xml nmap-bootstrap.xsl -o 192.168.0_Full_TCP.html
```

3. Cut your quick scan outputs into several hosts list by service:
Use the awesome https://github.com/ernw/nmap-parse-output.
You can list every discovered services:
```
$ ./nmap-parse-output 192.168.0_Quick.xml service-names
```

If [these services](guidelines/internal/ServicesEnumeration.md) are present, create a host list to launch enumerating or pwning attacks:
```
$ ./nmap-parse-output 192.168.0_Quick.xml http-ports > hosts_www.lst
$ ./nmap-parse-output 192.168.0_Quick.xml service ftp > hosts_ftp.lst
$ ./nmap-parse-output 192.168.0_Quick.xml service ssh > hosts_ssh.lst
$ ./nmap-parse-output 192.168.0_Quick.xml service telnet > hosts_telnet.lst
[...]
$ ./nmap-parse-output 192.168.0_Quick.xml service vnc > hosts_vnc.lst
```

4. Interesting projects:
* https://github.com/ernw/nmap-parse-output
* https://github.com/codingo/Reconnoitre

## Results analysis
### General overview
1. Is the network flat ?
2. Can you access to any assets ?
3. Can you access to sensitive services ?

### Services enumeration
Now it's time to enumerate and hopefully pwn [these services](guidelines/internal/ServicesEnumeration.md)...
