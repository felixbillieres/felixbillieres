---
description: https://app.hackthebox.com/machines/Authority
---

# 🧑‍⚖️ Authority

<figure><img src="../../.gitbook/assets/image (1017).png" alt=""><figcaption></figcaption></figure>

this is the output of the nmap:

<figure><img src="../../.gitbook/assets/image (1019).png" alt=""><figcaption></figcaption></figure>

We can connect to RPC via the following command:

```
rpcclient -U '' 10.129.229.56
```

<figure><img src="../../.gitbook/assets/image (1020).png" alt=""><figcaption></figcaption></figure>

And we can enumerate the shares, which will reveal we have read access on some shares ->

```
smbmap -H 10.129.229.56 -u null
```

<figure><img src="../../.gitbook/assets/image (1021).png" alt=""><figcaption></figcaption></figure>

So looking at the content of the Development file, we can find some interesting stuff:

```
smbmap -H 10.129.229.56 -u null -R 'Development'
```

<figure><img src="../../.gitbook/assets/image (1022).png" alt=""><figcaption></figcaption></figure>

We see some Ansible yml scripts

let's connect to navigate better:

```
smbclient -N //10.129.229.56/Development
```

After some enumeration we find the PWN subfile called ansible\_inventory

<figure><img src="../../.gitbook/assets/image (1023).png" alt=""><figcaption></figcaption></figure>

That could be evil-winrm creds but no it does not pass, we can verify with netexec->

```
netexec winrm authority.htb -u administrator -p 'Welcome1'
```

<figure><img src="../../.gitbook/assets/image (1024).png" alt=""><figcaption></figcaption></figure>

So we continue enumeration and find stuff in the \Automation\Ansible\PWM\defaults\ file

we get the main.yml file and look at the content:

<figure><img src="../../.gitbook/assets/image (1025).png" alt=""><figcaption></figcaption></figure>

So after looking it up we see some tool called ansible2john ->

