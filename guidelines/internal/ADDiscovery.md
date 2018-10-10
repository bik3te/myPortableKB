# Windows (AD) environment discovery
What is incredible with AD environment is that anything can be enumerated and analyzed. Every piece of information is stored within the attributes of an object and can be queried by a user with a domain account. It is possible to gather many information regarding:
* users;
* groups;
* computers;
* permissions;
* authentication protocols;
* password policy;
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
1. OUs
2. Groups
3. Users
4. Computers
### Trusts
### Passwords / hashes
### Password policy
### Risky configurations
1. Password never expires or is not required
2. Smartcard use but no password change
3. No Kerberos preauthentification required
4. User with adminCount attribute set to true
5. User trusted to authenticate for delegation
6. Null sessions or SMBv1
### Old Operating Systems
