# ❇️ Linux Local Password Attacks

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
grep -rnw "PRIVATE KEY" /home/* 2>/dev/null | grep ":1
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
