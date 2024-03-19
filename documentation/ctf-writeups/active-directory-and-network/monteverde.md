---
description: https://app.hackthebox.com/machines/Monteverde
---

# 🐭 Monteverde

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

So let's start by a quick nmap:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption><p>TCP DNS (53) along with Kerberos (TCP 88) and LDAP (TCP 389) suggests this might be a domain controller</p></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

So we found the name of some of the users and their account name

a good way to go would be to check if the users use their name as passwords for their account

So let's try to put all the usernames in a word list and spray using crackmapexec

```
crackmapexec smb 10.129.69.81 -u users.txt -p users.txt --continue-on-success
```

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption><p>MEGABANK.LOCAL\SABatchJobs:SABatchJobs</p></figcaption></figure>

cool we got a match&#x20;

Now let's use `smbmap` to tell me what shares there are and what I have access to in a cleaned up manner:

```
smbmap -H 10.129.69.81 -u SABatchJobs -p SABatchJobs
```

* `smbmap`: This is a command-line tool used to enumerate and interact with shares on SMB servers. It allows users to discover accessible shares, permissions, and other information available on SMB-enabled systems.
* `-H 10.129.69.81`: This parameter specifies the target host or IP address (`10.129.69.81`) of the SMB server that the `smbmap` tool will interact with. This is the remote system you want to analyze.

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

after some enumeration we see some interesting files:

```
smbmap -H 10.129.69.81 -u SABatchJobs -p SABatchJobs -R 'users$'
```

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

To grab it using smbclient ->

```
smbclient -U SABatchJobs //10.129.69.81/users$ SABatchJobs -c 'get mhope/azure.xml azure.xml'
```

1. `smbclient`: This is the command-line tool used to access files and services on SMB/CIFS servers.
2. `-U SABatchJobs`: This parameter specifies the username (`SABatchJobs`) to be used for authentication when connecting to the SMB/CIFS server.
3. `//10.10.10.172/users$`: This is the Uniform Naming Convention (UNC) path to the shared resource on the server (`10.10.10.172`). The `users$` represents a hidden share named `users`. The double slashes `//` indicate the beginning of a UNC path.
4. `SABatchJobs`: This is the password corresponding to the username provided with the `-U` flag. It's used for authentication purposes.
5. `-c 'get mhope/azure.xml azure.xml'`: This part of the command specifies a subcommand to be executed after connecting to the SMB/CIFS server. In this case, it's executing the `get` command to retrieve a file named `azure.xml` from the `mhope` directory on the server's `users$` share. The file will be downloaded to the local system and saved as `azure.xml`.

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Or another way of doing it:

```
smbclient -U SABatchJobs //10.129.69.81/users$ SABatchJobs
```

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

in the XML file, there are credentials:

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

We also get the indication that winrm works for this account so let's go:

```
evil-winrm -i 10.129.69.81 -u mhope -p '4n0therD4y@n0th3r$'
```

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption><p>59f4dfa90d416cb495bd38d04f361549</p></figcaption></figure>
