---
description: https://app.hackthebox.com/machines/PC
---

# 💻 PC

<figure><img src="../../.gitbook/assets/image (795).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (797).png" alt=""><figcaption></figcaption></figure>

After running a nmap we only got ssh and unknown service up:

<figure><img src="../../.gitbook/assets/image (798).png" alt=""><figcaption></figcaption></figure>

By connecting to the service using `nc`, you can observe its response. This can help you understand what kind of service is running on the specified port (in this case, port 50051) and potentially gather information about its behavior.

<figure><img src="../../.gitbook/assets/image (799).png" alt=""><figcaption></figcaption></figure>

The presence of non-readable characters suggests that the data being transmitted is not in a human-readable format, which is typical for binary data, encrypted data, or data encoded in a specific format.

<figure><img src="../../.gitbook/assets/image (800).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://book.hacktricks.xyz/pentesting-web/grpc-web-pentest" %}
