---
description: À apprendre par cœur
---

# ⛈️ Questions about AD

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption><p>Stay Hard</p></figcaption></figure>

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
