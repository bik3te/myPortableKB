# Windows (AD) environment discovery
What is incredible with AD environment is that anything can be enumerated and analyzed. Every piece of information is stored within the attributes of an object and can be queried by a user with a domain account. It is possible to gather many information regarding:
* the structure of the directory (way of classifying OUs / assets / etc.);
* users;
* groups;
* computers;
* permissions;
* authentication protocols;
* password policy;
* use of LAPS;
* GPOs and login scripts;
* delegations;
* domain trusts;
* etc.

## Tools
Obtaining the list of users, groups and computers is an easy task with the Windows [Net commands](../../tools/NetCommands.md).
However, it is possible to learn so much more with tools like:
* [ADExplorer](../../tools/ADExplorer.md);
* [PingCastle](../../tools/PingCastle.md);
* [PowerView](../../tools/PowerView.md).

## Interesting things to look for
### Privileged accounts
1. Privileged accounts with a password that never expires
2. Privileged accounts that can be delegated
3. Service accounts in a domain admin group -> [Kerberoasting](Kerberoasting.md)
4. Accounts with the "adminCount" attribute set to "1" -> They are / were in protected groups and may be used as [backoors](AdminSDHolder.md)
### Groups
1. Authenticated Users or Everyone present in a restricted group
### Computers
1. Old and vulnerable systems
### Trusts
1. Unknown domains in SIDHistory
2. Unknown account in delegation
3. Dangerous rights in delegation
### Passwords / hashes
1. Password do not expire
2. Password is not required
3. Password can be empty
4. [Reversible](GPPPasswords.md) passwords
5. Smartcard use but no password change
### Password policy
1. Complexity
2. Max password age
3. Min password age
4. Min password length
5. Password history
6. Lockout threshold
7. Lockout duration
8. Reset account counter locker after ?
9. Reversible encryption ?
### Risky configurations
1. Accounts without Kerberos preauthentification required -> [AS-REPRoasting](AS-REPRoasting.md)
2. DC allowing Null sessions or SMBv1
3. Weak LSA settings
- LSAAnonymousNameLookup
- RestrictAnonymous
- RestrictAnonymousSam
- LmCompatibilityLevel
4. SIDHistory creation is enabled
