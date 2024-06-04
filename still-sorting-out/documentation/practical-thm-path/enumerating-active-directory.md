---
description: https://tryhackme.com/r/room/adenumeration
---

# 🧑‍🍳 Enumerating Active Directory

<figure><img src="../../../.gitbook/assets/image (793).png" alt=""><figcaption></figcaption></figure>

Found AD credentials but nowhere to log in with them? Runas is the tool.

If we have the AD credentials in the format of :, we can use Runas, a legitimate Windows binary, to inject the credentials into memory.

```
runas.exe /netonly /user:<domain>\<username> cmd.exe
```

* /netonly - Since we are not domain-joined, we want to load the credentials for network authentication but not authenticate against a domain controller. So commands executed locally on the computer will run in the context of your standard Windows account, but any network connections will occur using the account specified here.
* /user - Here, we provide the details of the domain and the username. It is always a safe bet to use the Fully Qualified Domain Name (FQDN) instead of just the NetBIOS name of the domain since this will help with resolution.

Note that since we added the /netonly parameter, the credentials will not be verified directly by a domain controller so that it will accept any password.

### IP vs Hostnames

`dir \\za.tryhackme.com\SYSVOL` and `dir \\<DC IP>\SYSVOL`

When we provide the **hostname**, network authentication will attempt first to perform **Kerberos** authentication. Since Kerberos authentication uses hostnames embedded in the tickets, if we provide the **IP** instead, we can force the authentication type to be **NTLM**.

### Using Injected Credentials

Now that we have injected our AD credentials into memory, with the /**netonly** option, all network communication will use these injected credentials for authentication. This includes all network communications of applications executed from that command prompt window.

<figure><img src="../../../.gitbook/assets/image (794).png" alt=""><figcaption></figcaption></figure>
