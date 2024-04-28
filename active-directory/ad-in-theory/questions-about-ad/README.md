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

**Other interesting groups:**

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

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Restricted admin enabled</p></figcaption></figure>

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

