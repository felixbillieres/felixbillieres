# 🦹‍♂️ Filter Evasion

A type of injection mitigation is utilizing blacklisted characters and words on the back-end to detect injection attempts and deny the request if any request contained them.

### Filter/WAF Detection

Let's take a boosted version of our host checker with thome mitigations ->

<figure><img src="../../../.gitbook/assets/image (1307).png" alt=""><figcaption></figcaption></figure>

This could be the back end code that checks for those chars:

```php
$blacklist = ['&', '|', ';', ...SNIP...];
foreach ($blacklist as $character) {
    if (strpos($_POST['ip'], $character) !== false) {
        echo "Invalid input";
    }
}
```

### Bypass Blacklisted Spaces

We see that the newline operators does not block our request:

<figure><img src="../../../.gitbook/assets/image (1308).png" alt=""><figcaption></figcaption></figure>

But with the whoami added, and the space encoded it still blocks:

<figure><img src="../../../.gitbook/assets/image (1309).png" alt=""><figcaption></figcaption></figure>

So let's get rid off whoami and check if the space is the char blocking:

<figure><img src="../../../.gitbook/assets/image (1310).png" alt=""><figcaption></figcaption></figure>

It's good to know that Using tabs (%09) instead of spaces is a technique that may work, as both Linux and Windows accept commands with tabs between arguments so let's try `127.0.0.1%0a%09`

<figure><img src="../../../.gitbook/assets/image (1311).png" alt=""><figcaption></figcaption></figure>

Let's try with another technique Using the ($IFS) Linux Environment Variable may also work since its default value is a space and a tab, which would work between command arguments. So, if we use `${IFS}` where the spaces should be, the variable should be automatically replaced with a space, and our command should work. `127.0.0.1%0a${IFS}`

<figure><img src="../../../.gitbook/assets/image (1312).png" alt=""><figcaption></figcaption></figure>

There are many other methods we can utilize to bypass space filters, we can use the `Bash Brace Expansion`

```shell-session
ElFelixi0@htb[/htb]$ {ls,-la}

total 0
drwxr-xr-x 1 21y4d 21y4d   0 Jul 13 07:37 .
drwxr-xr-x 1 21y4d 21y4d   0 Jul 13 13:01 ..
```

_**Use what you learned in this section to execute the command 'ls -la'. What is the size of the 'index.php' file?**_

<figure><img src="../../../.gitbook/assets/image (1313).png" alt=""><figcaption><p>127.0.0.1%0a{ls,-la}</p></figcaption></figure>

### Other Blacklisted Characters

### Linux

`${IFS}` is a magical environment variable but there are not that for other interesting chars, let's get creative and use the variables we know to get our chars through the filters ->

```shell-session
ElFelixi0@htb[/htb]$ echo ${PATH}

/usr/local/bin:/usr/bin:/bin:/usr/games
```

So if we print our&#x20;

```shell-session
ElFelixi0@htb[/htb]$ echo ${PATH:0:1}

/
```

And we can't forget `$HOME` or `$PWD`

```shell-session
ElFelixi0@htb[/htb]$ echo ${LS_COLORS:10:1}

;
```

<figure><img src="../../../.gitbook/assets/image (1314).png" alt=""><figcaption></figcaption></figure>
