# 🍚 Parameter Fuzzing

## Parameter Fuzzing - GET

If after enumerating for directories we encounter something like this:

<figure><img src="../../../.gitbook/assets/image (1288).png" alt=""><figcaption></figcaption></figure>

That indicates that there must be something that identifies users to verify whether they have access to read the `flag`

Since we don't have cookies or a login form, maybe there is a key that we can pass to the page to read the `flag`

Let's see `GET` requests, which are usually passed right after the URL, with a `?` symbol, like:

* `http://admin.academy.htb:PORT/admin/admin.php?param1=key`.

```shell-session
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx
```

Which will lead to this page once we find the good parameter:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**Using what you learned in this section, run a parameter fuzzing scan on this page. what is the parameter accepted by this webpage?**_

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:50581/admin/admin.php?FUZZ=key -fs 798
```

<figure><img src="../../../.gitbook/assets/image (1394).png" alt=""><figcaption></figcaption></figure>

## Parameter Fuzzing - POST

The difference between `POST` requests and `GET` requests is that `POST` requests are not passed with the URL and cannot simply be appended after a `?` symbol. `POST` requests are passed in the `data` field within the HTTP request. To do this we can use the `-d` flag

```shell-session
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

and once we found stuff, we can send data using curl ->

```shell-session
curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'
```

## Value Fuzzing

In order to create custom wordlists, we can use the following command for numbers&#x20;

```shell-session
for i in $(seq 1 1000); do echo $i >> ids.txt; done
```

And to launch our value attack with custom wordlist:

```shell-session
ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

_**Try to create the 'ids.txt' wordlist, identify the accepted value with a fuzzing scan, and then use it in a 'POST' request with 'curl' to collect the flag. What is the content of the flag?**_

```
ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:50581/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs 768
```

```
curl http://admin.academy.htb:50581/admin/admin.php -X POST -d 'id=73' -H 'Content-Type: application/x-www-form-urlencoded'
```

<figure><img src="../../../.gitbook/assets/image (1395).png" alt=""><figcaption></figcaption></figure>
