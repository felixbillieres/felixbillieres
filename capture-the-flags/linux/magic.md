---
description: https://app.hackthebox.com/machines/Magic
---

# 🧙 Magic

<figure><img src="../../.gitbook/assets/image (1026).png" alt=""><figcaption></figcaption></figure>

So we got a web page with a login page that is easily bypassed by a SQLI:

{% embed url="https://book.hacktricks.xyz/pentesting-web/login-bypass#sql-injection-authentication-bypass" %}

<figure><img src="../../.gitbook/assets/image (1027).png" alt=""><figcaption></figcaption></figure>

We then try to upload PHP reverse shell:

<figure><img src="../../.gitbook/assets/image (1028).png" alt=""><figcaption></figcaption></figure>

But we get this pop up:

<figure><img src="../../.gitbook/assets/image (1029).png" alt=""><figcaption></figcaption></figure>

And when I try to upload JPG on PNG file, i have this one ?

<figure><img src="../../.gitbook/assets/image (1030).png" alt=""><figcaption></figcaption></figure>
