# Internal pentest

## Simple internal pentest

### If a laptop and a domain user account are provided
* [Quick Windows assessment](guidelines/internal/QuickWindowsAssessment.md)

### Plug your laptop on the network
1. Grab autoconfig information (DHCP)
```
$ ip addr && ip route && cat /etc/resolv.conf
```
2. Perform [Windows quick wins](guidelines/internal/WindowsQuickWins.md) and [network discovery](guidelines/internal/NetworkDiscovery.md) concurrently, it may sometimes be sufficient to get DA :)
3. Perform [Windows (AD) environment discovery](guidelines/internal/ADDiscovery.md)
4. [Pwn the domain](guidelines/internal/ADPwning.md)
5. During that time and when the network scans are finished, select interesting subnets (if not all) and perform a vulnerability scan with [Nessus](tools/Nessus.md). If you find vulns, use your brain and google to [think about the risks](guidelines/internal/SploitFromWeb.md), see with the client if it's okay to perform a [sploit attempt](tools/Metasploit.md) of some vulns on some hosts
6. Also during that time (you're a talented polyvalent and multitasked person!) check and pwn the servers (if possible) where [admin services](guidelines/internal/AdminServices.md) were found !
7. You can also perform [Bruteforce / dictionary attacks](guidelines/internal/Bruteforce.md), [Password spraying](guidelines/internal/PasswordSpraying.md) or [Hash cracking](guidelines/internal/PasswordCracking.md)
8. Do not forget to simulate [data exfiltration](guidelines/internal/DataExfiltration.md)

## Harder?
1. You may need to perform [MitM](guidelines/internal/Man-in-the-Middle.md)
2. Or to [bypass defenses](guidelines/internal/DefensesBypass.md)

## Specific sections
### Lateral Movement
* [Reverse shell guide](guidelines/internal/ReverseShell.md)
* [How to pivot with class](guidelines/internal/Pivoting.md)
* [How to share files between targets and attacking host](guidelines/internal/Infiltration.md)

### 1337 tricks

## VoIP

## Wi-Fi
