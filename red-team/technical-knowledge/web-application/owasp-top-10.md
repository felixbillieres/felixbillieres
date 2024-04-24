---
description: https://app.hackthebox.com/tracks/OWASP-Top-10
---

# 🐝 OWASP Top 10

<figure><img src="../../../.gitbook/assets/image (21) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Looking glass

First, let's see what the website does with simplesubmit:

<figure><img src="../../../.gitbook/assets/image (22) (1).png" alt=""><figcaption></figcaption></figure>

since it's very simple ill try the basic separator/command RCE ->

<figure><img src="../../../.gitbook/assets/image (23) (1).png" alt=""><figcaption><p>it works</p></figcaption></figure>

Let's try to dig more:

```
10.30.12.231; ls -al ../
```

<figure><img src="../../../.gitbook/assets/image (24) (1).png" alt=""><figcaption></figcaption></figure>

that's probably it ->

```
10.30.12.231; cat  ../flag_Sw2ZZ
```

So not too much to think about here, onto next one&#x20;

<figure><img src="../../../.gitbook/assets/image (25) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (26) (1).png" alt=""><figcaption></figcaption></figure>

### sanitize

:joy:

<figure><img src="../../../.gitbook/assets/image (27) (1).png" alt=""><figcaption></figcaption></figure>

happy to have my basics :joy:

### baby auth

Register as whatever username, get your session ID token and go to Cyberchef to decode it from base64-> change your name parameter to admin and encode it to base64 again

<figure><img src="../../../.gitbook/assets/image (29) (1).png" alt=""><figcaption></figcaption></figure>

Then change your session ID in inspect mode and enjoy:

<figure><img src="../../../.gitbook/assets/image (28) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (30) (1).png" alt=""><figcaption></figcaption></figure>
