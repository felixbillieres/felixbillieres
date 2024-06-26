---
description: https://app.hackthebox.com/machines/235
---

# 💦 Cascade

<figure><img src="../../.gitbook/assets/image (1170).png" alt=""><figcaption></figcaption></figure>

&#x20;We'll start with our nmap scan ->

<figure><img src="../../.gitbook/assets/image (1171).png" alt=""><figcaption></figcaption></figure>

Then we'll enumerate users with rpcclient:

```
rpcclient -U '' -N 10.129.35.57
```

<figure><img src="../../.gitbook/assets/image (1172).png" alt=""><figcaption></figcaption></figure>

Here is an awk command that will list out all the usernames:

```
awk -F'[][]' '{ print $2 }' users.txt
```

<figure><img src="../../.gitbook/assets/image (1173).png" alt=""><figcaption></figcaption></figure>
