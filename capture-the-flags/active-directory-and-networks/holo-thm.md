---
description: https://tryhackme.com/r/room/hololive
---

# 👻 HOLO (thm)

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So first I look at the network

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

then I scan the range that is given to us:

```
scope of the engagement is 10.200.x.0/24 
```

So we discover a webpage that is fueled by wordpress version 5.5.3:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we see a www.holo.live button so we can consider this is a valid host to add in /etc/hosts and when we reload the page we have this:&#x20;

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we launch our gobuster:

```
gobuster vhost -u holo.live -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomai\u2502
ns-top1million-110000.txt
```

dir mode is used to discover directories and files on a web server.

And vhost is designed to discover virtual hosts (subdomains) on a web server.

**How It Works**: Gobuster makes requests to the target server with different subdomain names (from a specified wordlist) in the `Host` header. It checks if the server responds differently for different subdomain names, indicating the presence of virtual hosts.

#### Key Differences

* **Target**: `dir` targets directories and files within a domain, while `vhost` targets subdomains or virtual hosts.
* **HTTP Requests**: In `dir` mode, the path component of the URL changes; in `vhost` mode, the `Host` header changes.
* **Discovery Goals**: `dir` mode is for finding hidden content within a single domain, whereas `vhost` mode is for discovering additional subdomains that might provide different content or services.

and find the following domains:

```
Found: dev.holo.live (Status: 200) [Size: 7515]
Found: admin.holo.live (Status: 200) [Size: 1845]
Found: www.holo.live (Status: 200) [Size: 21405] 
```

so we add them:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

feroxbuster is not installed, so now we need to discover subdomains on the different domains

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
feroxbuster -u http://DOMAIN.holo.live -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php -s 200 -d 1
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We discover some interesting stuff such as this img.php, probably used to load images through GET/POST requests:

So we capture the request of the dev.holo.live/img.php ->

And we are able to test for LFI:

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So maybe we can access to the supersecret file ->

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>admin:DBManagerLogin!</p></figcaption></figure>

