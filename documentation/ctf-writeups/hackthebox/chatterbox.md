---
description: https://app.hackthebox.com/machines/123
---

# 🐱 Chatterbox

<figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

Nmap -sC -sV did not work as well as usual, so i tried:&#x20;

```
nmap -T4 -A -p- 10.129.209.32
```

* `-T4`: This option sets the timing template for the scan. It specifies the timing aggression level, with higher levels increasing the speed of the scan. In this case, `-T4` sets it to level 4, which is a relatively fast scanning speed.
* `-A`: This option enables aggressive scanning. It includes various advanced options like OS detection, version detection, script scanning, and traceroute. This can provide more detailed information about the target.
* `-p-`: This option instructs Nmap to scan all 65,535 TCP ports on the target. By default, Nmap only scans the most common 1,000 ports, but using `-p-` ensures that every port is scanned.

still a problem because of tcpwrapped:

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://security.stackexchange.com/questions/23407/how-to-bypass-tcpwrapped-with-nmap-scan" %}

"TCPwrapped" is a term you might encounter when using network scanning tools like Nmap. It refers to a situation where a port responds with the status "tcpwrapped" during a scan. This status doesn't provide specific details about the service or application running on that port. Instead, it indicates that the port is protected by a TCP wrapper.

To bypass this I looked at the port and decided to only scan this one individually:

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

```
nmap -sV -p 9255 10.129.209.32
```

after doing a searchsploit achat we find a Buffer Overflow exploit, let's download it and look at it :

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

we have to modify the command given to us:

<figure><img src="../../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

run it in another tab:

<figure><img src="../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

copy the generated payload:

<figure><img src="../../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

and replace it in the python file:

<figure><img src="../../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

now modify the UDP socket to match our server:

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

launch a nc on port 8080 ( re did a payload with port 8080 because 443 did not work):

```
nc -nlvp 8080
```

and if in another tab you go and&#x20;

```
python payload.py
```

you get a shell

***

