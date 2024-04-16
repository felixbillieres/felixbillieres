---
description: >-
  https://portswigger.net/web-security/learning-paths/file-upload-vulnerabilities
---

# 📑 File Upload vulnerabilities

File upload vulnerabilities are when a web server allows users to upload files to its filesystem without sufficiently validating things like their name, type, contents, or size. Failing to properly enforce restrictions on these could mean that even a basic image upload function can be used to upload arbitrary and potentially dangerous files instead. This could even include server-side script files that enable remote code execution.

We already did some File uploads in the&#x20;

{% content-ref url="server-side-vulnerabilities.md" %}
[server-side-vulnerabilities.md](server-side-vulnerabilities.md)
{% endcontent-ref %}

### Lab: Web shell upload via path traversal

We start by uploading our php payload and capturing the request:

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

We saw that we were able to upload the PHP file, but it was as a txt file, the folder must filter out the PHP files, so let's save the PHP file in another directory. If we do a filename ../webshell.php it cuts the ../ but if we encode it to ..%2fwebshell.php it seems to work:

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

so we see when encoding it saves the shell in another directory ->

so i took te req and forwarded it, on the page it showed:

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

and now we go back on repeater, and we get the file:

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

### Lab: Web shell upload via extension blacklist bypass

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
