# 🎤 Miscellaneous Misconfigurations

#### Exchange Related Group

&#x20;The group `Exchange Windows Permissions` is not listed as a protected group, but members are granted the ability to write a DACL to the domain object. This can be leveraged to give a user DCSync privileges. An attacker can add accounts to this group by leveraging a DACL misconfiguration (possible) or by leveraging a compromised account that is a member of the Account Operators group. It is common to find user accounts and even computers as members of this group.&#x20;

The Exchange group `Organization Management` is another extremely powerful group (effectively the "Domain Admins" of Exchange) and can access the mailboxes of all domain users

#### PrivExchange

The `PrivExchange` attack results from a flaw in the Exchange Server `PushSubscription` feature, which allows any domain user with a mailbox to force the Exchange server to authenticate to any host provided by the client over HTTP.

The Exchange service runs as SYSTEM and is over-privileged by default (i.e., has WriteDacl privileges). This flaw can be leveraged to relay to LDAP and dump the domain NTDS database.

#### Printer Bug

The Printer Bug is a flaw in the MS-RPRN protocol (Print System Remote Protocol),  To leverage this flaw, any domain user can connect to the spool's named pipe with the `RpcOpenPrinter` method and use the `RpcRemoteFindFirstPrinterChangeNotificationEx` method, and force the server to authenticate to any host provided by the client over SMB.

The spooler service runs as SYSTEM and is installed by default in Windows servers running Desktop Experience. This attack can be leveraged to relay to LDAP and grant your attacker account DCSync privileges to retrieve all password hashes from AD.

**Enumerating for this printer bug ->**

```powershell
PS C:\htb> Import-Module .\SecurityAssessment.ps1
PS C:\htb> Get-SpoolStatus -ComputerName ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```

#### MS14-068

This was a flaw in the Kerberos protocol, which could be leveraged along with standard domain user credentials to elevate privileges to Domain Admin.  It can be exploited with tools such as the [Python Kerberos Exploitation Kit (PyKEK)](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS14-068/pykek) or the Impacket toolkit. The machine [Mantis](https://app.hackthebox.com/machines/98) showcases this vulnerability.

#### Sniffing LDAP Credentials

Many applications and printers store LDAP credentials in their web admin console to connect to the domain. These consoles are often left with weak or default passwords.  Other times, the application has a `test connection` function that we can use to gather credentials by changing the LDAP IP address to that of our attack host and setting up a `netcat` listener on LDAP port 389. Good idea to look at this  [post](https://grimhacker.com/2018/03/09/just-a-printer/).

#### Enumerating DNS Records

We can use a tool such as [adidnsdump](https://github.com/dirkjanm/adidnsdump) to enumerate all DNS records in a domain using a valid domain user account. . If all servers and workstations have a non-descriptive name, it makes it difficult for us to know what exactly to attack. If we can access DNS entries in AD, we can potentially discover interesting DNS records that point to this same server which we can use to better plan out our attacks.

**adidnsdump**

```shell-session
ElFelixi0@htb[/htb]$ adidnsdump -u inlanefreight\\forend ldap://172.16.5.5 
```

**View content of records.csv file**

```shell-session
ElFelixi0@htb[/htb]$ head records.csv 
```
