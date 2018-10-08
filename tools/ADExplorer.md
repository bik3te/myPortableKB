# AD Explorer
AD Explorer is an awesome tool to query domain controllers and ask it to enumerate it for you.
You only need one domain account.

## Execution
1. Copy it to the system
2. Download / run it from https://live.sysinternals.com/ADExplorer.exe
3. Load it into memory from Microsoft's site via UNC path:
Type **\\live.sysinternals.com\Tools\ADExplorer.exe** within the Run box or Explorer!

## Profit
You can now easily browse the directory in order to find interesting OUs, groups, computers, users, password policy, etc.

## Nice tricks
1. You can take a snapshot in order to perform your discovery without anymore querying DCs or running [Net commands](../../tools/NetCommands.md):
* From GUI: **FILE/Create Snapshot...**
* From command line:
```
> ADExplorer.exe -snapshot "" <domain_snapshot.dat>
> \\live.sysinternals.com\Tools\ADExplorer.exe -snapshot "" <domain_snapshot.dat>
```

2. Hunt for passwords
You should look for the following LDAP attributes:
- userPassword
- unicodePwd
- unixUserPassword
- comments
They can be populated with cleartext or "encoded" passwords (ASCII decimal format).
You should look if there are no other special attributes with *Pwd* or *Password* in it!
**Search/Search Container...**
```
  -> Class: User -- user
  -> Attribute: sAMAccountName
  -> Relation: not empty
  -> Add

  -> Class: User -- user
  -> Attribute: PwdLastSet
  -> Relation: not empty
  -> Add

  -> Class: User -- user
  -> Attribute: [userPassword / unicodePwd / unixUserPassword / comments / etc.]
  -> Relation: not empty
  -> Add
```
You also can save your research to gain time... :)
If you find an encoded password, just use a script like this one to get the cleartext:
```
>>> ''.join([chr(int(_)) for _ in '66 84 86 80 ...'.split()])
```
