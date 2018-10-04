# Quick Windows assessment

The aim is to quickly audit the Windows configuration, dump hashes and/or
attempt to crack them.

## Boot security

Reboot the system and try to access to the BIOS/UEFI configuration panel. Is
there a password to protect its access or alteration ? If no, it is possible to
boot on Linux from a USB stick, an external hard drive, a CD-ROM or the network
(PXE).

## Hard drive security

The hard drive may be protected/locked (only compatible with the initial laptop) or
encrypted. If it is not encrypted, it is possible to mount, browse, alter the
file system and dump Windows hashes from Linux. If the boot was protected but
the hard drive was not locked, it is possible to extract it from the laptop and
analyze it from a USB / HDD (SATA/IDE) adaptor.

## Gaining access to the system with privs from Live(CD/USB)

### Kon-Boot

Kon-Boot is an application which will silently bypass the authentication process
of Windows based operating systems. Without overwriting the old password. It is
possible to use it by booting from a USB key or a CD-ROM:
[Kon-Boot](https://www.piotrbania.com/all/kon-boot/)

### sethc.exe

Boot from a Linux live CD or a repair disk and replace
`C:\Windows\System32\sethc.exe` with `C:\Windows\System32\cmd.exe`. When the
sticky key combination (Shift 5 times) is pressed at the logon screen, you will
get access to a command prompt with NT-SYSTEM privileges.

### utilman.exe

Boot from a Linux live CD or a repair disk and replace
`C:\Windows\System32\utilman.exe` with `C:\Windows\System32\cmd.exe`. When
pressing `Windows key + U` at the logon screen, you will get access to a
command prompt with NT-SYSTEM privileges.

## Gaining access to the system with privs from DMA attacks

* [Inception](https://github.com/carmaa/inception)
* [PCILeech](https://github.com/ufrisk/pcileech)
* [Generating signatures](https://sysdream.com/news/lab/2017-12-22-windows-dma-attacks-gaining-system-shells-using-a-generic-patch/)

## Gaining access to the system with privs from PXE

1. PXE booting a Windows image with Hyper-V, VMWare or VirtualBox
```
Boot from Legacy Network Adapter
```

2. During the PXE boot deployment process, installation, or login steps, suspend or take a snapshot of the VM’s current state

3. Browse to the snapshot file location and look for the corresponding files for your hypervisor
- VMWare: .vmem, .vmsn (snapshot memory file), .vmss (suspended memory file)
- Hyper-V: .BIN, .VSV, .VMRS (virtual machine runtime file)

4. Grep and strings fu, example:
```
strings PXEtest.VMRS | grep -C 4 "UserID=deployment"
```

5. Change the boot mode and choose to boot from hard drive
```
Boot from IDE
```

6. Restart the VM and press Shift+F10 during the initial Windows setup process

7. Within the shell, create a user with administrator privileges as above

## Creating a user with administrator privileges (not stealthy!)

From the above methods:
```
net user audit_user /add
net localgroup administrators audit_user /add
```

## Performing a quick system safety-checks
* [Seatbelt](../../tools/Seatbelt.md)

## Escalating privileges
* PsExec:
```
PsExec.exe -i -s cmd.exe
```

* [Privilege escalation](PrivEsc.md)

## Dumping hashes

Dumping Windows hashes is a crucial part of penetration tests. It can be used in
order to perform [pass-the-hash](Pass-the-hash.md),
[password cracking](PasswordCracking.md),
[password policy analysis](PasspolAuditing.md), etc.

### Dumping SAM from linux

```
# mkdir -p /mnt/sda1
# mount /dev/sda1 /mnt/sda1
# bkhive /mnt/sda1/Windows/System32/config/SYSTEM /tmp/saved-syskey.txt
# samdump2 /mnt/sda1/Windows/System32/config/SAM /tmp/saved-syskey.txt > /tmp/hashes.txt
```

### Dumping SAM from Windows

```
reg save HKLM\SAM sam.hive
reg save HKLM\SECURITY security.hive
reg save HKLM\SYSTEM syskey
```

And one time again on Linux:
```
# bkhive syskey /tmp/saved-syskey.txt
# samdump2 sam /tmp/saved-syskey.txt > /tmp/hashes.txt
```

### Moar ?
* To check other [hashdump techniques](HashDump.md)
* To obtain more information about Windows hashes: [Windows hashes](WindowsHashes.md)
