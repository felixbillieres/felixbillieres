---
description: À apprendre par cœur
---

# ⛈️ Questions about AD

### Game Plan, 4 week AD rush

Week 1: Understanding the Basics

* Milestone 1 (End of Week 1):
  * Have a general understanding of the following sections:
    * Domains
    * Trusts
    * Users
    * Groups
    * Computers
    * Authentication
    * Authorization
  * Be able to explain the basic concepts of each section.

Week 2: Deepening Knowledge

* Milestone 2 (End of Week 2):
  * Have a detailed understanding of the following sections:
    * Trust types
    * Trust key
    * User properties
    * Important groups
    * Windows computers connection
    * NTLM
    * ACLs
  * Be able to describe the processes and mechanisms involved in each section.

Week 3: Mastering Advanced Concepts

* Milestone 3 (End of Week 3):
  * Have mastery over the following sections:
    * Trust transitivity
    * How to query the database?
    * Kerberos
    * Kerberos Delegation
    * Powershell remoting
    * SSH tunneling
  * Be able to explain advanced concepts and apply them to specific scenarios.

Week 4: Review and Consolidation

* Milestone 4 (End of Week 4):
  * Review all articles and sections to consolidate knowledge.
  * Practice exercises or simulations to test your understanding.
  * Be capable of explaining each article in detail and answering specific questions on each topic.

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption><p>Stay Hard</p></figcaption></figure>

**What is an AD?**

* It's a system that allows to manage a set of Computers & Users connected in the same network from a central server&#x20;

### Domain

**What is a Domain?**

* A domain is a set of computers connected to the same AD database that is managed by the same central server called **Domain Controller**

**What is a Domain Name?**

* every domain has a DNS name, it can be the same as their website (contoso.com) or and internal domain (contoso.local)

```
#current user domain 
$env:USERDNSDOMAIN
#current computer domain
(Get-WmiObject Win32_ComputerSystem).Domain
```

**What is a NetBIOS name?**

* facilitates communication between computers on a local area network (LAN). Every domain can have it's NetBIOS name. In the logs, you can identify users with `CONTOSO\Administrator`, where the first part is the NetBIOS name and the second one is the username.

**What is a Forest?**

* A forest is a tree of domains. The name of the forest is the name of the root domain (ex: contoso.com) and it has one or more subdomains:

```
              contoso.local
                    |
            .-------'--------.
            |                |
            |                |
     it.contoso.local hr.contoso.local
            | 
            |
            |
  webs.it.contoso.local
```

* each domain has its own database and its own Domain Controllers. Users from a domain can access ressources in another domain but not in another forest

**What are Functional Modes?**

* A functional mode is the version of a domain/forest. The modes are named based on the minimum Windows Server operative system required to work with them. Then if, for example, you find a domain/forest with Windows2012 mode, you can know that all the Domain Controllers are at least Windows Server 2012.

```
 #Get the mode of the forest/domain
(Get-ADForest).ForestMode
#OR
(Get-ADDomain).DomainMode
```

### **Trust**

**What are Trusts?**

* Trust is what links users and allow them to access to other domains in the same forest.  [A trust is a connection from a domain to another](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/cc731335\(v=ws.10\)). Not a physical network connection, but a kind of authentication/authorization connection.

**What is trust direction?**

* A trust direction is a one sided trust that allows the trusted users to access the trusted domains resources

```
 (trusting)         trusts        (trusted)
  Domain A  -------------------->  Domain B
       outgoing               incoming
       outbound               inbound
                    access
            <--------------------
```

**Difference between inbound and outbound connection:**

* If the trust is directed towards your current domain, it's an incoming (also called inbound) connection. If it goes from your domain to the other, it's an outcoming (also called outbound) connection.

```
#Trust of your current domain
nltest /domain_trusts
```

**What is Trust transitivity?**

* A [trust can be transitive or nontransitive](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/cc754612\(v=ws.10\)), a non-transitive trust indicates that the trust is not extended beyond the immediate relationship. In other words, just because entity A trusts entity B and entity B trusts entity C, it does not mean that entity A automatically trusts entity C whereas transitive trust means that trust is passed from one entity to another through a chain of trust relationships. Entity A trusts entity B, and entity B trusts entity C, then entity A also implicitly trusts entity C

transitive trust:

```
       (trusting)   trusts   (trusted)  (trusting)   trusts   (trusted)
  Domain A  ------------------->  Domain B --------------------> Domain C
                    access                          access
            <-------------------           <--------------------
```

* In a domain, to be able to access all the resources, you need all the parents and children to be connected by a bidirectional transitive trust:

```
              contoso.local
               ^  v   v  ^  
          .----'  |   |  '----.
          |  .----'   '----.  |
          ^  v             v  ^
     it.contoso.local hr.contoso.local
          ^  v 
          |  |
          ^  v
  webs.it.contoso.local
```

**What are the different Trust types?**

* **Parent-Child**: The default trusts created between a parent domain and its child.
* **Forest**: A trust to share resources between forests. This way any domain of the forest can access to any domain on the other forest (if the direction and transitivity of the trust allow it). If a forest trust is misconfigured, then it can allow to [take control of the other forest](http://www.harmj0y.net/blog/redteaming/not-a-security-boundary-breaking-forest-trusts/).
* **External**: A trust to connect to a specific domain that is in a non trusted forest.
* **Realm**: A special trust to connect Active Directory and a non-Windows domain.
* **Shortcut**: When two domains within the forest communicate often but are not directly connected, you can avoid jumping over many trusts by creating a direct shortcut trust.

**Explain Trust Keys:**

When you communicate through trust, there is a communication between the domain controller of your domain and the domain controller of the target domain and the domain controllers needs to share a key to keep the communications secure.

### Users

**Explain User Identifiers;**

* User identifiers are what we use to distinguish users. it can be the username (stored in the **SamAccountName** attribute) or the **SID** (a combination of the domain SID plus the user RID (Relative Identifier), which is the last number that appears in the user SID)

**What is User Secrets**?

*   For the Domain Controller to be able to authenticate the user, the database stores the sensitive data but not in plaintext. The following secrets are saved:

    * NT hash (and LM hash for the older accounts)
      * The hashes are stored both in Windows local [SAM](https://en.wikipedia.org/wiki/Security\_Account\_Manager) and Active Directory NTDS databases
    * Kerberos keys

    you need administrator privileges (or equivalent) to [dump the domain database](https://zer1t0.gitlab.io/posts/attacking\_ad/#domain-database-dumping) with a dcsync attack or grabbing the `C:\Windows\NTDS\ntds.dit` file from the Domain Controller.

**What can you do with those hashes?**

*   LM hashes are not used a lot because often obsolete. NT hashes can be used to perform [Pass-The-Hash](https://zer1t0.gitlab.io/posts/attacking\_ad/#pass-the-hash) or [Overpass-the-Hash](https://zer1t0.gitlab.io/posts/attacking\_ad/#pass-the-key) attacks in order to impersonate users in remote machines.

    Additionally, you can try to crack the LM and NT hashes with [hashcat](https://hashcat.net/) to recover the original password. If you are lucky and the LM hash is present, this should be quickly.

**What can you do with kerberos keys?**

* The keys can be used to ask for a Kerberos ticket that represents the user in Kerberos authentication. It can be used in a [Pass-The-Key](https://zer1t0.gitlab.io/posts/attacking\_ad/#pass-the-key) attack to retrieve a ticket for the impersonated user.

**What is UserAccountControl?**

* It's a proprety in the user class tha contains a **series of flags** that are very relevant for the security and the domain and used in many attacks (flags: [https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties))

**What are some Important Users and how to list them?**

```
net user /domain
#or
Get-ADUser -Filter * | select SamAccountName
```

* `Administrator` user is the most privileged account of the domain, `krbtgt` account is very important too. Its secrets (NT hash and Kerberos keys) are used to encrypt the tickets (specifically the TGTs) used by Kerberos that allows to authenticate users. If you are able to compromise the `krbtgt` account, you will be able of creating [Golden Tickets](https://en.hackndo.com/kerberos-silver-golden-tickets/).

**What are Computer accounts and difference with user accounts?**

* The difference between user accounts and computers accounts is that the firsts are stored as instances of [User class](https://docs.microsoft.com/en-us/windows/win32/adschema/c-user) in the database whereas the others are stored as instances of [Computer class](https://docs.microsoft.com/en-us/windows/win32/adschema/c-computer) (which is a subclass of User class). Moreover the computer accounts names are the computer hostname finished with a dollar sign `$`.

```powershell
PS C:\> Get-ADObject -LDAPFilter "objectClass=User" -Properties SamAccountName | select SamAccountName

SamAccountName
--------------
Administrator
Guest
DC01$
krbtgt
anakin
WS01-10$
WS02-7$
DC02$
han
POKEMON$
```

**What are Trust accounts?**

* It's good to know that you can have a normal user that finishes with a $, but when a trust is established, an associated user object is created in each domain to store the trust key. The name of the user is the NetBIOS name of the other domain, finished in $
* For example, in case of the trust between the domains FOO and BAR, the FOO domain would store the trust key in the BAR$ user, and the BAR domain would store it in the FOO$ user.

### Groups

**What are groups?**

* Rather than managing each of the users file permissions, you put them in groups  to handle the policy of all the users in a same group. the groups are stored in the domain database. And, in the same way, they can be identified by the `SamAccountName or the SID`

```
Get-ADGroup -Filter * | select SamAccountName
```

**Explain Administrative groups:**

* There are many [default groups](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#default-security-groups) defined for different roles in the domain/forest. As attacker, one of the most juicy groups is the [Domain Admins](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-domainadmins) group, that gives administrator privileges to its members in the domain

```
Get-ADGroup "Domain Admins" -Properties members,memberof
```

**What are other interesting groups:**

* The [Enterprise Admins](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-entadmins) group, which provides administrator privileges in all the forest. It's a group that only exists in the root domain of the forest, but is added by default to the [Administrators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#administrators) group of the all the domains in the forest. There is also the the `Domain Admins` group that is added to the `Administrators` group of the domain, as well as the `Administrators` groups of the domain computers.

```
                        .------------------------.
                        |     contoso.local      |
       .-------------------------------------------------------------.
       |                                                             |
       |                   .----------------.                        |  
       |               .-->| Administrators |<-.   .->Administrators |
       |               |   '----------------'  |   |     ____        | 
       |               |    .---------------.  |   |    |    |       |
       |               |    | Domain Admins |>-'---'    |____|       |
       |               |    '---------------'           /::::/       |
       |               |   .-------------------.                     |
       |               '--<| Enterprise Admins |                     |
       |                   '-------------------'                     |
       |                             v v                             |
       '-----------------------------|-|-----------------------------'  
                           |         | |      |                         
                           |         | |      |                         
                 .---------'         | |      '-----------.             
                 |                   v v                  |             
.----------------------------------. | | .----------------------------------.
|        it.contoso.local          | | | |        hr.contoso.local          |
|----------------------------------| | | |----------------------------------|
|                                  | v v |                                  |
|        .----------------.        | | | |        .----------------.        |
|     .->| Administrators |<---------' '--------->| Administrators |<-.     |
|     |  '----------------'        |     |        '----------------'  |     |
|     |  .---------------.         |     |        .---------------.   |     |
|     '-<| Domain Admins |         |     |        | Domain Admins |>--'     |
|        '---------------'         |     |        '---------------'         |
|                |                 |     |                |                 |
|        .-------'---------.       |     |        .-------'---------.       |
|        |                 |       |     |        |                 |       |
|        v                 v       |     |        v                 v       |
| Administrators    Administrators |     | Administrators    Administrators |
|       ____              ____     |     |      ____              ____      |
|      |    |            |    |    |     |     |    |            |    |     |
|      |____|            |____|    |     |     |____|            |____|     |
|      /::::/            /::::/    |     |     /::::/            /::::/     |
'----------------------------------'     '----------------------------------'
```

* other [important groups](https://adsecurity.org/?p=3700) to be taken into account

**What are different Group Scopes?**

* There are 3 different groups based on their scope:
  * **Universal** groups can include members from any domain within a forest or trusted forests and grant permissions within the same forest or trusted forests. The Enterprise Admins group is an example of a Universal group.
  * **Global** groups can include members only from the same domain and grant permissions within domains of the same forest or trusting domains or forests. The Domain Admins group is an example of a Global group.
  * **DomainLocal** groups can include members from the domain or any trusted domain and grant permissions only within their domains. The Administrators group is an example of a DomainLocal group.

### Computers <a href="#computers" id="computers"></a>

**Different types of Computers within a domain:**

* **Domain Controllers**: The central servers that manage the domain. They are Windows Server machines.
* **Workstations**: The personal computers used by people every day. These machines are usually Windows 10 or 7 machines.
* **Servers**: The computers that offers services such as webs, files or databases. They are usually Linux or Windows Server machines.

**What is a domain controller?**

* The domain controller is the **central server of a domain**, that is running the [Active Directory Domain Service](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) (AD DS). It keeps the domain DB stored in ths ntds.dit file.

**What is Domain database dumping?**

* When we become domain admin, in order to read sensitive files like krbtgt user creds or create Golden tickets, we can dump the DB of the DC&#x20;
* One way of doing this is [dumping the NTDS.dit file](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/ntds.dit-enumeration#no-credentials-ntdsutil) locally with [ntdsutil](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc753343\(v=ws.11\)) or [vssadmin](https://docs.microsoft.com/en-gb/windows-server/administration/windows-commands/vssadmin) or remote with the [impacket secretsdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py) script.

```
secretsdump.py 'contoso.local/Administrator@192.168.100.2' -just-dc-user krbtgt
```

**How to discover Windows computers in a domain or network?**

* ([NetBIOS name service](https://zer1t0.gitlab.io/posts/attacking\_ad/#netbios-name-service) listens in the port 137) We can scan the network and perform a NetBIOS scan by using a tool like [nbtscan](http://www.unixwiz.net/tools/nbtscan.html) or nmap [nbtstat](https://nmap.org/nsedoc/scripts/nbstat.html) script.
* ([SMB](https://zer1t0.gitlab.io/posts/attacking\_ad/#smb) listens in the port 445) We can take advantage of the NTLM authentication negotiation to retrieve the machine name. You can perform an scan with [ntlm-info](https://github.com/Zer1t0/ntlm-info) or nmap [smb-os-discovery](https://nmap.org/nsedoc/scripts/smb-os-discovery.html) script.

#### How to connect with RPC/SMB

* If you have port 445 open, you can use tools such as [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec) and the impacket examples [psexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py), [wmiexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py) ([wmiexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py) will also need the port 135)
* You can also perform Pass-the-hash with the NT or LM hash directly with impacket tools:

```
psexec.py contoso.local/Anakin@192.168.100.10 -hashes :cdeae556dc28c24b5b7b14e9df5b6e21
```

* With the PsExec tool, [inject the NT hash in the Windows session with mimikatz](https://stealthbits.com/blog/passing-the-hash-with-mimikatz/).

With those techniques we connect via NTLM auth, but by default Kerberos is used and we would need to provide a kerberos tickets with those tools

* in Windows we will need to [inject the ticket in the session](https://gist.github.com/TarlogicSecurity/2f221924fef8c14a1d8e29f3cb5c5c4a#using-ticket-in-windows) by using [mimikatz](https://github.com/gentilkiwi/mimikatz) or [Rubeus](https://github.com/GhostPack/Rubeus).

Windows and linux use different ticket formay that we can convert using [ticket\_converter](https://github.com/Zer1t0/ticket\_converter) or [cerbero](https://github.com/Zer1t0/cerbero#convert).

#### How to use **Powershell Remoting to connect**

to connect to a Windows machine and get a Powershell session in the remote machine

* From windows we need to use [CmdLets and parameters](https://docs.microsoft.com/en-us/powershell/scripting/learn/ps101/08-powershell-remoting?view=powershell-7.1)
  * if we have a Kerberos ticket or a NT hash, we will need to inject them by using [Rubeus](https://github.com/GhostPack/Rubeus) or [mimikatz](https://github.com/gentilkiwi/mimikatz).

```
.\Rubeus.exe asktgt /user:Administrator /rc4:b73fdfe10e87b4ca5c0d957f81de6863 /ptt

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v1.6.1

[*] Action: Ask TGT

[*] Using rc4_hmac hash: b73fdfe10e87b4ca5c0d957f81de6863
[*] Building AS-REQ (w/ preauth) for: 'contoso.local\Administrator'
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFQjCCBT6gAwIBBaEDAgEWooIETzCCBEthggRHMIIEQ6ADAgEFoQ8bDUNPTlRPU08uTE9DQUyiIjAg
      oAMCAQKhGTAXGwZrcmJ0Z3QbDWNvbnRvc28ubG9jYWyjggQFMIIEAaADAgESoQMCAQKiggPzBIID7xK3
      <!--stripped-->
      ERgPMjAyMTA1MDgwMjQzMjZapxEYDzIwMjEwNTE0MTY0MzI2WqgPGw1DT05UT1NPLkxPQ0FMqSIwIKAD
      AgECoRkwFxsGa3JidGd0Gw1jb250b3NvLmxvY2Fs
[+] Ticket successfully imported!

  ServiceName           :  krbtgt/contoso.local
  ServiceRealm          :  CONTOSO.LOCAL
  UserName              :  Administrator
  UserRealm             :  CONTOSO.LOCAL
  StartTime             :  07/05/2021 18:43:26
  EndTime               :  08/05/2021 04:43:26
  RenewTill             :  14/05/2021 18:43:26
  Flags                 :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType               :  rc4_hmac
  Base64(key)           :  95a1NmgYXwOmiyCa3qlplA==

PS C:\> Enter-PSSession -ComputerName dc01
[dc01]: PS C:\Users\Administrator\Documents> whoami
contoso\administrator
[dc01]: PS C:\Users\Administrator\Documents> hostname
dc01
[dc01]:
```

* From a Linux machine you can use [evil-winrm](https://github.com/Hackplayers/evil-winrm).

{% content-ref url="../../../interacting-with-protocols-and-tools/tools/winrm.md" %}
[winrm.md](../../../interacting-with-protocols-and-tools/tools/winrm.md)
{% endcontent-ref %}

#### How to connect with RDP

* From windows we can use [mstsc](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/mstsc)
* From Linux we can use [rdesktop](http://www.rdesktop.org/), [freerdp](https://www.freerdp.com/) or [remmina](https://remmina.org/)
  * Unlike RPC/SMB, RDP transmits the password in plain text and allows the remote computer to cache the user's credentials. it enables the user to log in to the remote system without having to enter their password again at each connection thus simulating the experience of being physically logged into their own machine. This makes it impossible to use pass-the-hash attacks
  * But there is a mode called Restricted Admin, when enabled, you don't send plain text so it's then possible to perform pass the hash connections.
    * From Linux, you can [use freerdp to perform a Pass-The-Hash with RDP](https://www.kali.org/blog/passing-hash-remote-desktop/)
    * And from Windows you can [inject a NT hash or Kerberos ticket with mimikatz or Rubeus and then use `mstsc.exe /restrictedadmin`](https://shellz.club/pass-the-hash-with-rdp-in-2019/)

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Restricted admin enabled</p></figcaption></figure>

#### **Windows computers credentials**

#### **Explain LSASS credentials**

* On windows machine -> common place to find creds -> LSASS Process (lsass.exe) in charge of user auth
* when user logs in -> credentials cached in lsass process in order to use SSO -> credentials caches by some [SSPs](https://zer1t0.gitlab.io/posts/attacking\_ad/#windows-ssps) (Security Support Providers) each one uses different auth methods, example:
  * The [Kerberos SSP](https://zer1t0.gitlab.io/posts/attacking\_ad/#kerberos-ssp)
  * The [Digest SSP](https://zer1t0.gitlab.io/posts/attacking\_ad/#digest-ssp)
  * The [NTLMSSP](https://zer1t0.gitlab.io/posts/attacking\_ad/#ntlm-ssp)
* We can extract LSASS credentials using [mimikatz](https://github.com/gentilkiwi/mimikatz). We can launch mimikatz directly in the target machine, or [dumping the LSASS memory](https://www.deepinstinct.com/2021/01/24/lsass-memory-dumps-are-stealthier-than-ever-before/) with some tool like [procdump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump), [comsvcs.dll](https://lolbas-project.github.io/lolbas/Libraries/Comsvcs/) or [werfault.exe](https://www.deepinstinct.com/2021/02/16/lsass-memory-dumps-are-stealthier-than-ever-before-part-2/) and then process the generated memory dump with mimikatz or [pypikatz](https://github.com/skelsec/pypykatz). can also to use [lsassy](https://github.com/Hackndo/lsassy) to read a dump remotely avoiding to have to download the entire memory dump, that can take several megabytes.

**Explain Registry credentials**

* Other location to find credentials is the registry. One of the places where sensible credentials are stored is in the [LSA secrets](https://passcape.com/index.php?section=docsys\&cmd=details\&id=23).
* In the LSA secrets you can find:
  * **Domain Computer Account:** To be part of the domain, the computer must have a user account in the domain with available username and password. They are stored in the LSA secrets&#x20;
  * **Service users passwords:** If you want to run some services as the user, the computer needs to store the password but it does not store the user
  * **Auto-logon password:** If windows [auto-logon is enabled](https://keithga.wordpress.com/2013/12/19/sysinternals-autologon-and-securely-encrypting-passwords/), the password can be stored in the LSA secrets. The other alternative is that it is saved in `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon` registry key under the key `DefaulUserName`.
  * **DPAPI master keys:** The [data protection API](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection) (DPAPI) is used to allow users encrypt sensible data without need to worry about cryptographic keys. If you are able to retrieve the master keys, then you [can decrypt users data](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/dpapi-extracting-passwords).

**How to dump registry credentials:**

To get the credentials from the SECURITY and SAM hives, we can read them from memory by using mimikatz.

execute `token::elevate` to acquire a `SYSTEM` session to read the credentials.&#x20;

`privilege::debug` if required to enable the SeDebugPrivilege.

other commands:

* `lsadump::secrets`: Get the LSA secrets.
* `lsadump::cache`: Retrieve the cached domain logons.
* `lsadump::sam`: Fetch the local account credentials.

We can also save a copy of the hive files with [reg save](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/reg-save) command, move them to our machine, and finally to get the content with [impacket secretsdump](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py) script or mimikatz.

#### Linux computers <a href="#linux-computers" id="linux-computers"></a>

**How to discover linux machines on a network?:**

Since Linux computers don't have any characteristic port opened by default, however many Linux machines are used as servers that are remotely administrate. In order to administrate Linux computers, usually the [SSH](https://zer1t0.gitlab.io/posts/attacking\_ad/#SSH) protocol is used. The SSH server service listens in the port 22

Moreover, older Linux machines may have [Telnet](https://en.wikipedia.org/wiki/Telnet) enabled on port 23.

**where to find linux computer credentials**

Linux machines usually have a Kerberos client that is configured with the domain computer account. You can find the credentials in the keytab (`/etc/krb5.keytab)`or specified in the [Kerberos configuration file](https://web.mit.edu/kerberos/krb5-1.12/doc/admin/conf\_files/krb5\_conf.html) in `/etc/krb5.conf.`

You can use the keys stored to [ask for a Kerberos ticket](https://www.tarlogic.com/en/blog/how-to-attack-kerberos/) an impersonate the user.

{% embed url="https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-active-directory#ccache-ticket-reuse-from-keytab" %}

**What can you say about linux kerberos tickets?**

* in order to be identified within the system, the linux machines has kerberos client configured  within the computer account. You can find the creds usaully in the `/etc/krb5.keytab` OR `/etc/krb.conf` file and you can display it using `klist` command
* the next step would be to impersonate the user in the domain since, when authentificated, a kerberos ticket is retrieved under the `/tmp` directory in files with the format `krb5cc_%{uid}` . it is also possible that tickets are stored in the [Linux kernel keys](https://man7.org/linux/man-pages/man7/keyrings.7.html) instead of files, but you can grab them and convert to files by using [tickey](https://github.com/TarlogicSecurity/tickey).

**Where are stored the ssh private keys?**

* usually they are stored in the .ssh directory in a file called `id_rsa`
* In case the private key doesn't require a passphrase for using it, then you may can use it to connect to another machines in the domain with this syntax:

```
$ ssh -i id_ed25519_foo_key foo@db.contoso.local
```

{% embed url="https://robertholdsworthsecurity.medium.com/how-to-crack-an-ssh-private-key-passphrase-ab7dd1583178" %}

* a place worth looking is the bash history located in the `.bash_history` file of the user directory to find some credentials or command history

### Services <a href="#services" id="services"></a>

**What is the difference between active directory service and computer service?**

A windows or linux machine service could be understood as a background process running like a task (ex: database) but it is not obliged for a service to be listening on a port:

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

it could simply be a service that for example checks for updates on the system

on the other hand, active directory services are identifieres that indicates what remote services are or can be available (listening on a port) on a machine. Not all the remote services are registered in the domain database, however, the registration is required for those services that need to authenticate domain users through Kerberos.

* each active directory service stores those informations:
  * The **user** that runs the computer service.
  * The [service class](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc772815\(v=ws.10\)#service-principal-names), that indicates what kind of service is, for example web servers are registered like www class.
  * The **machine** where the service is hosted.

Since all services dont require to be executed and are stored in AD database, it's good to know that old services can lead to account takeover using kerberoast&#x20;

### Database <a href="#database" id="database"></a>

So we know the basics of the database of the domain and some objects stored in it like users, groups etc..

**Can you go more in depth on classes and maybe subclasses?**

We know that there is the [User class](https://docs.microsoft.com/en-us/windows/win32/adschema/c-user), the [Computer class](https://docs.microsoft.com/en-us/windows/win32/adschema/c-computer) or the [Group class](https://docs.microsoft.com/en-us/windows/win32/adschema/c-group).

But a class can have a **subclasse** that allows to inherit propreties. For example, the Computer class is a subclass of User class, therefore the computer objects can have the same properties of the user objects ,like `SAMAccountName`, and some new custom properties,

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

All the classes are subclasses of the [Top](https://docs.microsoft.com/en-us/windows/win32/adschema/c-top) class & many of the most relevant classes when performing a pentest, like User and Group, are attached to [Security-Principal](https://docs.microsoft.com/en-us/windows/win32/adschema/c-securityprincipal) auxiliary class, the class that defines the `SAMAccountName` and `SID` properties.

**What do you know about properties of a class?**&#x20;

Usually, properties are stored as a string value like _name_ or something else. There are some properties that contain sensitive data that are marked as [confidential properties](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/mark-attribute-as-confidential)

for example, the [UserPassword](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-adts/f3adda9f-89e1-4340-a3f2-1f0a6249f1f8) and [UnicodePwd](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-ada3/71e64720-be27-463f-9cc5-117f4bc849e1) propreties cannot be read, only written. When a [password change](https://docs.microsoft.com/en-US/troubleshoot/windows/win32/change-windows-active-directory-user-password) is required, these properties can be written in order to modify the user password.

Only authorized users can retrieve the confidential properties.&#x20;

There are some properties that require to meet certain conditions before being written.

**Define principals in active directory?**

In Active Directory, a **principal is a security entity**. The most common **principals are users, groups and computers**.

{% embed url="https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-principals" %}

**What are 3 kinds of SIDs in active directory?**

**Domain SID:** used to identify the domain, as well as the base for SIDs of the domain principals.

**Principal SID:** used to identify principals. It is compose by the domain SID and a principal RID

[**Well-known SIDs**](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/security-identifiers-in-windows) that identify abstract entities for special situations. Some of them are:

* **Authenticated Users**
* **Principal Self**
* **Administrator**
* **Domain Admins**

**What are Distinguished Names?**

The distinguished name is like a path that indicates the object position in the database hierarchy

It's used to identify objects in the db and reference objects (ex. the members of a group are referenced by its `DistinguishedName`.)

It's composed by:

* Domain Component (DC) -> for `it.domain.com` the DC part will be `DC=it,DC=domain,DC=com`.
* [Organizational Unit](https://en.wikipedia.org/wiki/Active\_Directory#Organizational\_units) (OU)
* Common Name (CN) -> in `CN=Administrator,CN=Users,DC=contoso,DC=local`, the `CN=Users` identifies the Users container.

**What does the term Partitions refer to in AD databases?**

The databases are divided by partitions:

* **Domain**: Stores the domain objects.
* **Configuration**: Stores configuration of the domain, such as the `HOST` service alias or `Well-known` SIDs that we have seen before.
* **Schema**: Stores the definition of the classes and properties used by the database.
* **Domain DNS Zones**: Stores the DNS records of the domain and subdomains.
* **Forest DNS Zones**: Stores the DNS records of the rest of the forest, including parent domains.

**How would you define Global Catalog?**

To speed up searches through objects in other domains, some domain controllers have a read-only partitions with a subset of objects of other domains. These Domains Controllers can be called [Global Catalogs](https://docs.microsoft.com/pt-pt/previous-versions/windows/server/cc737410\(v=ws.10\)#domain-controller-and-global-catalog-server-structure)

**Explain querying with LDAP & ADWS**

To be able to interract with the domain controller database and it's data, we wan use different protocoles/services ->

**LDAP** allows to filter objects, interact and edit them in the db via a query syntax and you can go more in depth and retrieve propreties from the objects you interract with like the name (can check out more on [LDAP wiki](https://ldapwiki.com/)). We can pretty much interract with anything that is not highly sensitive. When enumerating active directory, ldap is a go to, you can use [ldapsearch](https://linux.die.net/man/1/ldapsearch) and [ldapmodify](https://linux.die.net/man/1/ldapmodify) tools (on linux). since we can modify objects we could imagine adding a user.

On the other hand there is **ADWS** (Active Directory Web Services), it also manipulates domain objects but it's based on [SOAP](https://en.wikipedia.org/wiki/SOAP) messages. It works very well with LDAP and is compatible with LDAP filters. So much that when used, the DC internally uses LDAP reqs to retrieve results

**Other protocols worth to know**

* The [DNS](https://en.wikipedia.org/wiki/Domain\_Name\_System) protocol
* The [SAMR](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/MS-SAMR/4df07fab-1bbc-452f-8e92-7853a3c7e380) (Security Account Manager Remote) protocol
* The [DRSR](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-drsr/f977faaa-673e-4f66-b9bf-48c640241d47) (Directory Replication Service Remote) protocol
* The [Kerberos](https://www.tarlogic.com/en/blog/how-kerberos-works/) authentication protocol
* The [Netlogon](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-nrpc/ff8f970f-3e37-40f7-bd4b-af7336e4792f) protocol

## Security

the 3 main topics in the security realm of Active Directory are:

**Address resolution; Authentication and Authentication**

### Address resolution <a href="#address-resolution" id="address-resolution"></a>

**What are some dangers related to misconfigured address resolution?**

The main danger is if a user or program can be tricked into connecting to an erroneous machine. Some common attacks are :

* Person-in-The-Middle (PitM) where the attacker interecpts the communication of the user and maybe interract with it if not encrypted
* [NTLM Relay](https://en.hackndo.com/ntlm-relay/) attack, using NTLM authentification from a victim and redirect it to a malicious server&#x20;
* [NTLM crack](https://0xdf.gitlab.io/2019/01/13/getting-net-ntlm-hases-from-windows.html) attack where the attacker just tries to crack the NTLM hash

**What are the 3 types of addresses that need to be resolved?**

**MAC address ->** unique identifier of each computer in the world. Sends messages via the ethernet protocol (can be altered with spoofing)

**IP address ->** used by the IP protocol on the internet layer. Allows communication via different computers in a same network (can read more about IP address attribution with [DHCP](https://zer1t0.gitlab.io/posts/attacking\_ad/#dhcp) or static IPs)

**Hostnames ->** Just like websites and DNS, Hostnames help us memorize and interract with computers with human-friendly names like `computerOfFelix` rather than just an IP that is hard to remember

**What are some vital processes for computers when it comes to finding the correct address to comunicate?**

**Hostname-IP resolution**

* to map a hostname to it's ip, computers can ask ask central server for hostname resolution which is used by DNS or they can send broadcast request asking all the computers to identify themselves (used by netBIOS, LLMNR, mDNS)

**IP-MAC resolution**

* &#x20;Once we find an IP, we can broadcast to all to tell the network card of the computer to identify itself by asking for the MAC address related to the IP using the ARP protocol

**IP configuration**

* Either it's done manually or dynamically using DHCP, we need to configure the IP of a computer. The danger if done dynamically is that the computer is not configured and asks blindly to the DHCP server using broadcast and can be tricked if an attacker has a fake DNS server

**What is ARP?**

[ARP](https://en.wikipedia.org/wiki/Address\_Resolution\_Protocol) (Address Resolution Protocol) allows to map the relation between the IP address of a computer and its MAC (Media Access Control) address.

the client machine sends t broadcast ARP request to the local network, asking for the one that has the target IP address. Then the computer with that IP should respond identifying its MAC. Finally the client sends the application packets to that Ethernet address.

```
                                                   .---.
                                                  /   /|
                                                 .---. |
                                       .-------> |   | '
                                       |         |   |/ 
                                       |         '---'  
   .---.                               |
  /   /|                               |           .---.
 .---. |    1) Who is 192.168.1.5?     |          /   /|
 |   | ' >-------->>-------------------.-------> .---. |
 |   |/                                |         |   | '
 '---'   <---------.                   |         |   |/ 
                   |                   |         '---'  
                   |                   |
                   |                   |           .---.
                   |                   |          /   /|
                   |                   '-------> .---. |
                   |                             |   | '
                   '-<<------------------------< |   |/ 
                     2)  I am 192.168.1.5        '---'  
                      (MAC 01:02:03:04:05:06)
```

**Explain ARP Spoofing**

An attacker could try to respond to all ARP requests to impersonate other computers. To add a layer of security, rather than sending ARP requests every time they communicate, computers keep the old responses in their ARP cache to reduce the number of requests needed.

**Explain how DHCP works**

DHCP helps configure dynamic IP addresses to the computers of a network. When new to a network, the foreign computer will look for the DHCP server to get a config that allows them to interract with the network, it works in 4 steps:

\-> Broadcast from the client to the server to find DHCP server

\-> Server responds to request with valid IP

\-> client receives IP and sends message to server to request it

\-> Server confirms client can use the IP and config params like IP renewal time

```
client        server
    |             |
    |  discovery  |
    | ----------> |
    |             |
    |    offer    |
    | <---------- |
    |             |
    |   request   |
    | ----------> |
    |             |
    | acknowledge |
    | <---------- |
    |             |
```

**What is a Rogue DHCP server**

A rogue DHCP server is a device on a network that, without permission, gives out IP addresses and network settings to devices, which can cause network problems and security risks. Therefore the perfect tool for an attacker could create a rogue DHCP server in order to set a custom configuration in the clients and redirect them to fake computers or domains controlled by the attacker

**Explain DHCP Starvation attack:**

It's a DOS attack where a fake client requests all the available IPs the DHCP server has to offer and make it impossible for legitimate users to obtain an IP:

```
$ dhcpstarv -i enp7s0
08:03:09 11/30/20: got address 192.168.100.7 for 00:16:36:99:be:21 from 192.168.100.2
08:03:09 11/30/20: got address 192.168.100.8 for 00:16:36:25:1f:1d from 192.168.100.2
08:03:09 11/30/20: got address 192.168.100.9 for 00:16:36:c7:79:f2 from 192.168.100.2
```

#### DNS <a href="#dns" id="dns"></a>

**What is DNS?**

DNS is a protocol that resolves DNS names of a computer to it's address (port 53). (it can also be used with mapping an IP to its name or resolving the aliases for a name)

```
    client                     DNS server
    .---.   A elFelixio.oscp?     .---.
   /   /| ------------------>  /   /|
  .---. |                     .---. |
  |   | '   185.199.111.153   |   | '
  |   |/  <------------------ |   |/ 
  '---'                       '---'
```

**What are DNS zones?**

DNS is hierarchical, each zone keeps record for it's domain and the subdomains attached to it. Let's take 2 zones for example:

_Zone contoso.com_

```
contoso.com
mail.contoso.com
www.contoso.com
```

_Zone internal.contoso.com_

```
internal.contoso.com
it.internal.contoso.com
admin.internal.contoso.com
hr.internal.contoso.com
```

Those are 2 independent zones, it makes it easier to keep track. The DNS server will comunicate with those zones in order to privide information to other zones

{% hint style="info" %}
With the `www.contoso.com` IP address, the DNS server needs to communicate with contoso authoritative DNS server, that manages the contoso.com zone, in order to retrieve this information.
{% endhint %}

**Explain DNS exfiltration:**

<figure><img src="../../../.gitbook/assets/image (1040).png" alt=""><figcaption></figcaption></figure>

Let's say we have a server that is isolated and has no internet access, but it can perform DNS queries. If the local DNS performs requests to other DNS servers on the internet, it can be abused: let's say i have the DNS server `malicious.com` , every query made to be or other subdomains will reach my server. An attacker could query a subdomain like `felix.malicious.com` and technically, the query should go through the parent domain, thus, retrivieng informations. (attacker can use a tool like [iodine](https://github.com/yarrick/iodine) or [dnscat2](https://github.com/iagox86/dnscat2))

**How can DNS zone transfer be dangerous?**

We know that zone transfer replicates all DNS server records. If misconfigured, anyone could perform zone transfers

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If just one DC allows to perform the zone transfer whereas the rest of DCs refuse the zone transfer, the misconfigured DCcould lead to anyone being able to perform zone transfers, thus recolecting all the DNS information without require any credentials.

#### **What to do if zone transfers aren't allowed?**

Since DNS records are stored in the AD database, we can read it using LDAP as any domain user to then dump all the DNS records, we can use [adidnsdump](https://github.com/dirkjanm/adidnsdump) to do so and it will save the output un _records.csv_ file

**What is ADIDNS?**

{% embed url="https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/adidns-spoofing" %}

**ADIDNS** is the Active Directory's implementation of DNS. In this setup, the role of the DNS server is assumed by the Domain Controller (DC). This is because the Active Directory database contains the DNS names of the computers within the domain as well as other DNS records. This integration allows for seamless name resolution and management of DNS entries within the Active Directory environment.

**What are DNS dynamic updates?**

Dynamic updates allows clients to create/modify/delete DNS records. Since any user can create a DNS record, the user that created the record becomes the owner of this record, thus, only authorized users of a certain record can update of modify/delete the record&#x20;

**How Could be abuse Dynamic updates?**

{% embed url="https://www.netspi.com/blog/technical-blog/network-penetration-testing/exploiting-adidns/" %}

{% embed url="https://www.netspi.com/blog/technical-blog/network-penetration-testing/adidns-revisited/" %}

Since DNS records are stored in the Active Directory database, they can be created/modify/deleted by using LDAP.

Registering a wildcard record (\*) could be interesting since it's used to specify a default IP address that is used to resolve those queries that doesn't match any other record. It could lead to [perform PitM attacks](https://blog.netspi.com/exploiting-adidns/) if it is used to point to a computer controlled by us.

#### What is NETBIOS?

[NetBIOS](https://en.wikipedia.org/wiki/NetBIOS) (Network Basic Input/Output System) helps applications in the same LAN (Local Area Network) communicate between them. Howerever it was not able to communicate them on different networks so this led to the creation of NBT protocol (NetBIOS over TCP/IP) to make NetBIOS work over TCP and UDP protocols and allow applications that used NetBIOS to communicate over internet.

#### What are the 3 services used by NETBIOS?

* **NetBIOS Datagram Service ->** used as transport layer for application protocols that requires a connectionless communication.
* **NetBIOS Session Service ->** used as transport for connected-oriented communications
* **NetBIOS Name Service ->** This one is very interesting for pentesting, it allows to:&#x20;
  * Resolve NetBIOS name to an IP address
  * Known the status of a NetBIOS node
  * Register/Release a NetBIOS name

#### What is the NBNS protocol and how does it work?

The NBNS was implemented as [WINS](https://web.archive.org/web/20031010135027/http://www.neohapsis.com:80/resources/wins.htm#sec4sub4sub4) (Windows Internet Name Service). In a network, each Windows computer has a WINS database that stores the available network resources, netbios and domain name.

So in order to resolve a NetBIOS name we can query using the WINS server or If this is not possible, then the query can be sent to the IP broadcast address, waiting for the answer from the target computer (but dangerous since anyone can respond, this is used by [responder.py](https://github.com/lgandx/Responder))

#### What is LLMNR?

{% content-ref url="../../attacking-vectors/methodology/initial-attack-vectors/llmnr-poisoning.md" %}
[llmnr-poisoning.md](../../attacking-vectors/methodology/initial-attack-vectors/llmnr-poisoning.md)
{% endcontent-ref %}

LLMNR is a descentralized application protocol that allows to resolve hostnames in the same local network. A common vuln with LLMNR is it is used to resolve names in local link by sending A DNS querie that anyone could repond to, responder.py also abuses this feature

#### What is mDNS?

[mDNS](https://en.wikipedia.org/wiki/Multicast\_DNS) (multicast DNS) is a descentralized application protocol, similar to LLMNR that allows to resolve names in local networks

#### What is WPAD?

The [WPAD](https://en.wikipedia.org/wiki/Web\_Proxy\_Auto-Discovery\_Protocol) (Web Proxy Auto-Discovery) is a protocol for browsers to get dynamically a file that indicates the proxies they should use.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**What is the difference between those protocols?**

While they all perform name resolution, the primary difference lies in their order of preference for resolving names:

1. **DNS (Domain Name System)**
2. **mDNS (Multicast DNS)**
3. **LLMNR (Link-Local Multicast Name Resolution)**
4. **NBNS (NetBIOS Name Service)**

Each protocol has a specific context and method for resolving names, which dictates its usage priority in network communications.

### Authentication <a href="#authentication" id="authentication"></a>
