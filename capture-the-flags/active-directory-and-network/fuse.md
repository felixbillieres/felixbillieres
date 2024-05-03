---
description: https://app.hackthebox.com/machines/Fuse
---

# ⚖️ Fuse

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

We first start by scanning the ports and adding everything to /etc/hosts

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

on port 80 we find this page:



<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

When clicking on the view buttons, we find some possible usernames:



we put it in a txt file and launch a userenum:

```
kerbrute userenum --dc 10.129.2.5 -d fabricorp.local users.txt
```

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

