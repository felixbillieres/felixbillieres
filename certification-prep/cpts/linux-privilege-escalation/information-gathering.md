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

<pre class="language-shell-session"><code class="lang-shell-session"><strong>cat /etc/passwd | cut -f1 -d:
</strong>
root
daemon
bin
sys

</code></pre>

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

<pre class="language-shell-session"><code class="lang-shell-session"><strong>find / -type d -name ".*" -ls 2>/dev/null
</strong></code></pre>

check all tmp files ->

```shell-session
ls -l /tmp /var/tmp /dev/shm
```

_**Enumerate the Linux environment and look for interesting files that might contain sensitive data. Submit the flag as the answer.**_



## Linux Services & Internals Enumeration

\
Here are some questions we can ask ourselves when enumerating:

* What services and applications are installed?
* What services are running?
* What sockets are in use?
* What users, admins, and groups exist on the system?
* Who is current logged in? What users recently logged in?
* What password policies, if any, are enforced on the host?
* Is the host joined to an Active Directory domain?
* What types of interesting information can we find in history, log, and backup files
* Which files have been modified recently and how often? Are there any interesting patterns in file modification that could indicate a cron job in use that we may be able to hijack?
* Current IP addressing information
* Anything interesting in the `/etc/hosts` file?
* Are there any interesting network connections to other systems in the internal network or even outside the network?
* What tools are installed on the system that we may be able to take advantage of? (Netcat, Perl, Python, Ruby, Nmap, tcpdump, gcc, etc.)
* Can we access the `bash_history` file for any users and can we uncover any thing interesting from their recorded command line history such as passwords?
* Are any Cron jobs running on the system that we may be able to hijack?

**Network Interfaces**

\
We can start to look at the network interfaces to see if we're dual homed with the following command ->

```
ip a
```

Then we can look in the /etc/hosts file ->

```shell-session
cat /etc/hosts
```

Then we can check out each user's last login time to try to see when users typically log in to the system and how frequently.

```shell-session
lastlog
```

It is also important to check a user's bash history, as they may be passing passwords as an argument on the command line, working with git repositories, setting up cron jobs, and more.

```shell-session
history
```

we can also find special history files created by scripts or programs. This can be found, among others, in scripts that monitor certain activities of users

```shell-session
find / -type f \( -name *_hist -o -name *_history \) -exec ls -l {} \; 2>/dev/null
```

It's also a good idea to check for any cron jobs on the system. Cron jobs on Linux systems are similar to Windows scheduled tasks.&#x20;

```shell-session
 ls -la /etc/cron.daily/
```

The [proc filesystem](https://man7.org/linux/man-pages/man5/proc.5.html) (`proc` / `procfs`) is a particular filesystem in Linux that contains information about system processes, hardware, and other system information. It is the primary way to access process information and can be used to view and modify kernel settings.&#x20;

```shell-session
find /proc -name cmdline -exec cat {} \; 2>/dev/null | tr " " "\n"
```

If it is a slightly older Linux system, the likelihood increases that we can find installed packages that may already have at least one vulnerability. to check installed packages ->

```shell-session
 apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list
```

always a classic to look at sudo version ->

```shell-session
sudo -V
```

Occasionally it can also happen that no direct packages are installed on the system but compiled programs in the form of binaries. Then we can hop on [GTFObins](https://gtfobins.github.io/) to see which one we can exploit

```shell-session
ls -l /bin /usr/bin/ /usr/sbin/
for i in $(curl -s https://gtfobins.github.io/ | html2text | cut -d" " -f1 | sed '/^[[:space:]]*$/d');do if grep -q "$i" installed_pkgs.list;then echo "Check GTFO for: $i";fi;done
```

We can use the diagnostic tool `strace` on Linux-based operating systems to track and analyze system calls and signal processing. It allows us to follow the flow of a program and understand how it accesses system resources, processes signals, and receives and sends data from the operating system

```shell-session
strace ping -c1 10.129.112.20
```

we can try to read almost all configuration files on a Linux operating system if the administrator has kept them the same.

```shell-session
/ -type f \( -name *.conf -o -name *.config \) -exec ls -l {} \; 2>/dev/null
```

To discover scripts ->&#x20;

```shell-session
find / -type f -name "*.sh" 2>/dev/null | grep -v "src\|snap\|share"
```

To check if it is a script created by the administrator in his path and whose rights have not been restricted, we can run it without going into the `root` directory.

```shell-session
ps aux | grep root
```

## Credential Hunting

When enumerating a system, it is important to note down any credentials. These may be found in configuration files (`.conf`, `.config`, `.xml`, etc.), shell scripts, a user's bash history file, backup (`.bak`) files, within database files or even in text files

The /var directory typically contains the web root for whatever web server is running on the host. The web root may contain database credentials or other types of credentials that can be leveraged to further access.

If we take for example a wordpress MYSQL db ->

```shell-session
cat wp-config.php | grep 'DB_USER\|DB_PASSWORD'
```

The spool or mail directories, if accessible, may also contain valuable information or even credentials.

```shell-session
find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null
```

### SSH Keys

&#x20;Whenever finding SSH keys check the `known_hosts` file to find targets. This file contains a list of public keys for all the hosts which the user has connected to in the past and may be useful for lateral movement or to find data on a remote host that can be used to perform privilege escalation on our target.

```shell-session
ls ~/.ssh
```
