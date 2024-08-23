# 🦥 Linux Privilege Escalation

### Enumeration

Good to start with [LinEnum](https://github.com/rebootuser/LinEnum) to look at Kernel versions, OS version, Running services etc...

To list the current processes we can run&#x20;

```shell-session
ps aux | grep root
```

We can also look for SSH keys that be used to access other systems or scripts and configuration files containing credentials.

```shell-session
ElFelixi0@htb[/htb]$ ls /home

backupsvc  bob.jones  cliff.moore  logger  mrb3n  shared  stacey.jenkins
```

We can check individual user directories and check to see if files such as the `.bash_history` file are readable and contain any interesting commands, look for configuration files, and check to see if we can obtain copies of a user's SSH keys.

Here is the ssh directory ->

```shell-session
ElFelixi0@htb[/htb]$ ls -l ~/.ssh

total 8
-rw------- 1 mrb3n mrb3n 1679 Aug 30 23:37 id_rsa
-rw-r--r-- 1 mrb3n mrb3n  393 Aug 30 23:37 id_rsa.pub
```

We also need to look if the user can run any commands either as another user or as root and look for `NOPASSWD`

```shell-session
sudo -l
```

Sometimes we can see password hashes directly in the /etc/passwd file so it's worth looking for it:

```shell-session
ElFelixi0@htb[/htb]$ cat /etc/passwd
 
<SNIP>
sysadm:$6$vdH7vuQIv6anIBWg$Ysk.UZzI7WxYUBYt8WRIWF0EzWlksOElDE0HLYinee38QI1A.0HW7WZCrUhZ9wwDz13bPpkTjNuRoUGYhwFE11:1007:1007::/home/sysadm:
```

We can also look for cron jobs to try and leverage to escalate privileges when the scheduled cron job runs->

```shell-session
ls -la /etc/cron.daily/
```

We have to look to see if we can mount an additional drive or unmounted file system, you may find sensitive files, passwords, or backups that can be leveraged to escalate privileges

```shell-session
ElFelixi0@htb[/htb]$ lsblk

NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   30G  0 disk 
├─sda1   8:1    0   29G  0 part /
├─sda2   8:2    0    1K  0 part 
└─sda5   8:5    0  975M  0 part [SWAP]
sr0     11:0    1  848M  0 rom  
```

Here is the command to find writeable directories ->

```shell-session
find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null
```

And here for writable files ->

```shell-session
 find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```
