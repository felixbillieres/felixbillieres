---
description: https://app.hackthebox.com/machines/Nibbles
---

# 🫧 Nibbles

<figure><img src="../../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

First let's look at the scan that reveals a webserver:

port 22 is open so let's go look at the source code of the page

<figure><img src="../../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

we find a directory, let's go and try to see some more directories:

```
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 10.129.211.182/nibbleblog
```

<figure><img src="../../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

ok start of the enum is going well ->

<figure><img src="../../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

while looking for nibbleblog CVEs we encounter [https://github.com/dix0nym/CVE-2015-6967](https://github.com/dix0nym/CVE-2015-6967)

we also finde a code execution solution that looks better: [https://packetstormsecurity.com/files/133425/NibbleBlog-4.0.3-Shell-Upload.html](https://packetstormsecurity.com/files/133425/NibbleBlog-4.0.3-Shell-Upload.html)

note that in the RCE, we can read:

<figure><img src="../../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

after looking for a bit we find some admin area:

<figure><img src="../../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>

let's try default credentials

and bingo, admin/nibbles were good:

<figure><img src="../../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

now let's go in the plugin area because earlier we read something about it:

<figure><img src="../../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

bingo we find a MyImage plugin with a downloadable area&#x20;

<figure><img src="../../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

let's go and fetch a reverse shell on [https://github.com/pentestmonkey/php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell)

set up a listener on port 4444 and change this code in your reverse shell php file:

<figure><img src="../../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

submit it and remember what was written in the exploit about the default path where is stored the file uploaded:

<figure><img src="../../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

```
http://10.129.211.182/nibbleblog/content/private/plugins/my_image/image.php
```

so i did this request to trigger the script and:

<figure><img src="../../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

now, we do a sudo -l, we found out we can run a file called monitor.sh as root:

<figure><img src="../../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

The file does not exist! check mate we can create a file that grants us root access via this file ->

create the file with a _**mkdir /home/nibbler/personal/stuff**_

<figure><img src="../../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

I didn't manage to use nano or vi so I did it manually

```
echo '#!/bin/sh' > filename.sh
echo 'bash' >> filename.sh
```

This series of commands creates a file named `filename.sh` and writes the specified content into it. The `>` operator overwrites the file if it exists, and `>>` appends to the file. Adjust the filename as needed.

don't forget to chmod +x and to run the file with sudo and then bam ->

<figure><img src="../../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (508).png" alt=""><figcaption></figcaption></figure>
