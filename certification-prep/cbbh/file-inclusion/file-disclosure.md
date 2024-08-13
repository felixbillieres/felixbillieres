# 🤽‍♀️ File Disclosure

## Local File Inclusion (LFI)

Let's see how we can exploit these vulnerabilities in different scenarios to be able to read the content of local files on the back-end server.

Imagine a web app where we can choose language ->

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

if the web application is pulling a file that is now being included in the page, we may be able to change the file being pulled to read the content of a different local file.

```
http://<SERVER_IP>:<PORT>/index.php?language=/etc/passwd
```

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Sometimes we will need to use relatives paths and going back to root ->

```
http://<SERVER_IP>:<PORT>/index.php?language=../../../../etc/passwd
```

A more advanced LFI attack is a `Second Order Attack`. This occurs because many web application functionalities may be insecurely pulling files from the back-end server based on user-controlled parameters.

For example, a web application may allow us to download our avatar through a URL like (`/profile/$username/avatar.png`). If we craft a malicious LFI username (e.g. `../../../etc/passwd`), then it may be possible to change the file being pulled to another local file on the server and grab it instead of our avatar.

_**Using the file inclusion find the name of a user on the system that starts with "b".**_

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

_**Submit the contents of the flag.txt file located in the /usr/share/flags directory.**_

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## Basic Bypasses
