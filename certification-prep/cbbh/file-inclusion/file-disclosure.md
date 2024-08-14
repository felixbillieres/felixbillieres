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

One common LFI filter is one that deletes substrings of (`../`), to bypass this if we use `....//` as our payload, then the filter would remove `../` and the output string would be `../` as well as `..././` or `....\/` and several other recursive LFI payloads. Furthermore, in some cases, escaping the forward slash character may also work to avoid path traversal filters (e.g. `....\/`), or adding extra forward slashes (e.g. `....////`)

Another way of bypassing would be to URL encode `../` into `%2e%2e%2f`, which may bypass the filter

<figure><img src="../../../.gitbook/assets/image (1380).png" alt=""><figcaption></figcaption></figure>

{% content-ref url="../command-injections/" %}
[command-injections](../command-injections/)
{% endcontent-ref %}

Some web applications may also use Regular Expressions to ensure that the file being included is under a specific path. Let's say we can call paths that are under the `./languages ,` we could start our payload with the approved path, and then use `../` to go back to the root directory

```
<SERVER_IP>:<PORT>/index.php?language=./languages/../../../../etc/passwd
```

***

some web applications append an extension to our input string (e.g. `.php`), to ensure that the file we include is in the expected extension.

In earlier versions of PHP, defined strings have a maximum length of 4096 characters

PHP also used to remove trailing slashes and single dots in path names, so if we call (`/etc/passwd/.`) then the `/.` would also be truncated, and PHP would call (`/etc/passwd`)

it is also important to note that we would also need to `start the path with a non-existing directory` for this technique to work.

```url
?language=non_existing_directory/../../../etc/passwd/./././.[./ REPEATED ~2048 times]
```

Here is a command to do that ->

```shell-session
echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done
```

***

PHP versions before 5.5 were vulnerable to `null byte injection`, which means that adding a null byte (`%00`) at the end of the string would terminate the string and not consider anything after it.

_**The above web application employs more than one filter to avoid LFI exploitation. Try to bypass these filters to read /flag.txt**_

```
http://94.237.54.205:42808/index.php?language=languages/....//....//....//....//flag.txt
```
