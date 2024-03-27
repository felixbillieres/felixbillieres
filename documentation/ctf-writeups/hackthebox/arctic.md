---
description: https://app.hackthebox.com/machines/9
---

# 🧊 Arctic

<figure><img src="../../../.gitbook/assets/image (686) (1).png" alt=""><figcaption></figcaption></figure>

After a Nmap →

<figure><img src="../../../.gitbook/assets/image (688).png" alt=""><figcaption></figcaption></figure>

In a subfile i found an admin login panel:

<figure><img src="../../../.gitbook/assets/image (689).png" alt=""><figcaption></figcaption></figure>

after looking for some exploits i found something interesting:

<figure><img src="../../../.gitbook/assets/image (690).png" alt=""><figcaption></figcaption></figure>

After looking at the payload, you can modify these:

<figure><img src="../../../.gitbook/assets/image (691).png" alt=""><figcaption></figcaption></figure>

run the exploit with a&#x20;

```
python 50057.py
```

You can start a listener or let the payload generate one for you, and see:

<figure><img src="../../../.gitbook/assets/image (692).png" alt=""><figcaption></figcaption></figure>

after some quick enumeration we found some interesting stuff:&#x20;

<figure><img src="../../../.gitbook/assets/image (693).png" alt=""><figcaption></figcaption></figure>

as well as&#x20;

<figure><img src="../../../.gitbook/assets/image (694).png" alt=""><figcaption></figcaption></figure>

unicorn - [https://github.com/trustedsec/unicorn.git](https://github.com/trustedsec/unicorn.git)
