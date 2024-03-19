---
description: https://app.hackthebox.com/machines/148
---

# ☢️ Active

<figure><img src="../../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

After our nmap we see an SMB share, after trying out a few things we spot the replication share has anonymous enabled&#x20;

<figure><img src="../../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

after a recursive search we find a Groups.xml file with interesting details

<figure><img src="../../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

after looking it up, we found this was a gpp file and to decrypt it was very easy:

{% embed url="https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/credential-access/unsecured-credentials/group-policy-preferences/gpp-password" %}

<figure><img src="../../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

now we need to access the share as SVC\_TGS user to see if we have more privileges&#x20;

```
smbclient -W 10.129.217.86 -U SVC_TGS //10.129.217.86/USERS
```

* `smbclient`: This is the command-line tool for interacting with SMB servers.
* `-W 10.129.217.86`: The `-W` option is used to specify the workgroup or domain. In this case, the workgroup or domain is set to "10.129.217.86."
* `-U SVC_TGS`: The `-U` option is used to specify the username for the authentication. In this case, the username is set to "SVC\_TGS."
* `//10.129.217.86/USERS`: This part specifies the target SMB server and the share or directory on that server. In this case:
  * `//10.129.217.86`: This is the IP address (or hostname) of the SMB server.
  * `/USERS`: This is the name of the shared resource or directory on the server that you are trying to access.

Now let's look for some juicy files:

<figure><img src="../../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

use john over hashcat for it to work:

```
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

<figure><img src="../../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

or another way to play it would've been to us GetUserSPN.py from impacket:

```
GetUserSPNs.py -request active.htb/SVC_TGS
```

<figure><img src="../../../.gitbook/assets/image (560).png" alt=""><figcaption></figcaption></figure>

put it in a hash.txt file and use john again:

```
john --wordlist=/usr/share/wordlists/rockyou.txt hashadmin.txt
```

<figure><img src="../../../.gitbook/assets/image (561).png" alt=""><figcaption></figcaption></figure>

Now we have 100 ways of going root but let's use psexec for this time:

```
psexec.py administrator@active.htbcd 
```

<figure><img src="../../../.gitbook/assets/image (562).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (563).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (564).png" alt=""><figcaption></figcaption></figure>
