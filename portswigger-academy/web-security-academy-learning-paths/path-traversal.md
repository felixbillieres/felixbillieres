---
description: https://portswigger.net/web-security/learning-paths/path-traversal
---

# 😋 Path traversal

Path traversal is also known as directory traversal. These vulnerabilities enable an attacker to read arbitrary files on the server that is running an application.

<figure><img src="../../.gitbook/assets/image (807).png" alt=""><figcaption></figcaption></figure>

### Lab: File path traversal, simple case

<figure><img src="../../.gitbook/assets/image (808).png" alt=""><figcaption></figcaption></figure>

So I open an image of the website in a tab and capture the request:

<figure><img src="../../.gitbook/assets/image (809).png" alt=""><figcaption></figcaption></figure>

and modify the request to access the passwd file:

<figure><img src="../../.gitbook/assets/image (810).png" alt=""><figcaption></figcaption></figure>

### Lab: File path traversal, traversal sequences blocked with absolute path bypass

The absolute path of my req bypassed their protection:

<figure><img src="../../.gitbook/assets/image (811).png" alt=""><figcaption></figcaption></figure>

### Lab: File path traversal, traversal sequences stripped non-recursively

captured the image again and doubled the args in the path ....//

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

### Lab: File path traversal, traversal sequences stripped with superfluous URL-decode

On this one we just had to double encode the request to bypass the filter:

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://www.urlencoder.org/" %}

this worked as well:

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

### Lab: File path traversal, validation of start of path

After capturing the req of an image, we see this odd path:

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

We get only 400 if we don't start fetching from /var/www/images:

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

example:

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

### Lab: File path traversal, validation of file extension with null byte bypass

we simply trick the request with the null byte %00:

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>
