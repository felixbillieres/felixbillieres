# 💎 Basic Fuzzing

## Directory Fuzzing

the main two options are `-w` for wordlists and `-u` for the URL, then we choose the keyword `FUZZ` to it by adding `:FUZZ` after it:

```shell-session
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ
```

We can even go faster with `-t 200`, but this is not recommended, especially when used on a remote site, as it may disrupt it, and cause a `Denial of Service`

## Page Fuzzing

we must find out what types of pages the website uses, like `.html`, `.aspx`, `.php`

We can start at looking at the pages header -> if the server is `apache`, then it may be `.php`, or if it was `IIS`, then it could be `.asp` or `.aspx`

The wordlist contains dots so we don't need to put it in our command ->

```shell-session
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ
```

And once we find some extensions that work, we can go and look to fuzz some directories ->

```shell-session
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php
```

## Recursive Fuzzing

In `ffuf`, we can enable recursive scanning with the `-recursion` flag, and we can specify the depth with the `-recursion-depth` flag. If we specify `-recursion-depth 1`, it will only fuzz the main directories and their direct sub-directories. we can specify our extension with `-e .php` and we can add the flag `-v` to output the full URLs

```shell-session
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v
```

_**Try to repeat what you learned so far to find more files/directories. One of them should give you a flag. What is the content of the flag?**_

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://94.237.59.199:35902/FUZZ -recursion -recursion-depth 1 -e .php -v
```

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
