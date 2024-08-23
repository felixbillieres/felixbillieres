# 🍫 Information Gathering

## Environment Enumeration

Here are some basic commands to get a better understanding of the machine ->

* `whoami`
* `id`
* `hostname`
* `ifconfig`
* `sudo -l`

Here is the command to chech our OS ->

```shell-session
cat /etc/os-release
```

Next step is to check out our current user's PATH, which is where the Linux system looks every time a command is executed for any executables to match the name of what we type

```shell-session
ElFelixi0@htb[/htb]$ echo $PATH

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

We also need to check out all environment variables that are set for our current user

```shell-session
ElFelixi0@htb[/htb]$ env

SHELL=/bin/bash
PWD=/home/htb-student
LOGNAME=htb-student
XDG_SESSION_TYPE=tty
MOTD_SHOWN=pam
HOME=/home/htb-student
LANG=en_US.UTF-8
```

Next we need to check if we're running a vulnerable Kernel version

```shell-session
uname -a
```

What login shells exist on the server

```shell-session
cat /etc/shells
```

we can take a look at the drives and any shares on the system. If we discover and can mount an additional drive or unmounted file system, we may find sensitive files, passwords, or backups

```shell-session
lsblk
```

The command `lpstat` can be used to find information about any printers attached to the system.

We can try  grepping for common words such as password, username, credential, etc in `/etc/fstab`

We can Check out the routing table by typing `route` or `netstat -rn`

to get all the users ->

```shell-session
cat /etc/passwd | cut -f1 -d:

root
daemon
bin
sys

```

check which users have login shells ->

```shell-session
ElFelixi0@htb[/htb]$ grep "*sh$" /etc/passwd

root:x:0:0:root:/root:/bin/bash
mrb3n:x:1000:1000:mrb3n:/home/mrb3n:/bin/bash
<SNIP>
```

View mounted file systems ->

```shell-session
df -h
```

view unmounted file systems:

```shell-session
cat /etc/fstab | grep -v "#" | column -t
```

check all hidden files ->

```shell-session
find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null | grep htb-student
```

All hidden directories ->

<pre class="language-shell-session"><code class="lang-shell-session"><strong>fœind / -type d -name ".*" -ls 2>/dev/null
</strong></code></pre>

check all tmp files ->

```shell-session
ls -l /tmp /var/tmp /dev/shm
```
