## Windows hashes

### Where to find them?

#### On workstations or servers

By dumping the Windows Security Accounts Manager (`SAM`) registry file. It stores
hashes of users' passwords. There are two formats:
* [LM](http://en.wikipedia.org/wiki/LM_hash)
* [NTLM](http://en.wikipedia.org/wiki/NTLM)

#### From a domain controller (AD)
By dumping the `NTDS.DIT` binary database file.

#### Within the LSASS process memory
By attaching to the LSASS process and playing with it like [Benjamin Delpy](https://twitter.com/gentilkiwi).

#### Through the network
By listening and abusing the network with a tool like
[Responder](../../tools/Responder.md) or [Inveigh](../../tools/Inveigh.md)

### How to crack them?

You can crack them with tools like [Hashcat](../../tools/Hashcat.md) or
[john](../../tools/john.md)

### Formats

#### LM

This format is so old and easily crackable. It was turned off by default since
Windows Vista/Server 2008.

#### NTLM (or NTHash)

This format is used in modern environment. It is harder to crack and can be used
to [pass-the-hash](Pass-the-hash.md) with tools like
[pth-toolkit](../../tools/pth-toolkit.md) or the [impacket suite](../../tools/impacket.md).

#### Net-NTLM (v1 or v2)

These formats are produced when a client and a server attempt to establish a
connection through the network (share, HTTP server, MS-SQL server, etc.): the
NTLM protocol uses the NTHash in a challenge/response between both devices. To
obtain a response to crack from the client, you can use
[Responder](../../tools/Responder.md) or [Inveigh](../../tools/Inveigh.md). If you don't have
enough time or can't crack it, you can perform [NTLM relaying](../../tools/Responder.md).
