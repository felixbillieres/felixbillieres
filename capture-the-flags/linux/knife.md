---
description: https://app.hackthebox.com/machines/Knife
---

# 🔪 Knife

<figure><img src="../../../.gitbook/assets/image (523).png" alt=""><figcaption></figcaption></figure>

We see a simple web server:

<figure><img src="../../../.gitbook/assets/image (524).png" alt=""><figcaption></figcaption></figure>

the go buster did not give anything, so let's see,

```
curl --head http://10.129.211.69
```

The `curl --head` command is used to retrieve only the headers of a specified URL using the HTTP HEAD method. In the provided command, `http://10.129.211.69` is the URL to which the HEAD request is being sent. This command is often used to check the response headers of a web server without downloading the actual content of the page. It can be useful for quickly inspecting server information, such as HTTP headers like server type, content type, and more.

<figure><img src="../../../.gitbook/assets/image (525).png" alt=""><figcaption></figcaption></figure>

we see the version of PHP running on this website&#x20;

after a quick search, we see a possible RCE:

<figure><img src="../../../.gitbook/assets/image (526).png" alt=""><figcaption></figcaption></figure>

```
searchsploit -m php/webapps/49933.py
```

and just like that we get a shell:

<figure><img src="../../../.gitbook/assets/image (527).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (528).png" alt=""><figcaption></figcaption></figure>

quick flag :tada:
