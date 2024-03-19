---
description: https://app.hackthebox.com/machines/225
---

# 🐦 Nest

<figure><img src="../../../../.gitbook/assets/image (495).png" alt=""><figcaption></figcaption></figure>

#### What's the name of the service according to `nmap` listening on port 4386? (Include the version in your answer)

<figure><img src="../../../../.gitbook/assets/image (497).png" alt=""><figcaption></figcaption></figure>

#### It turns out that Guest authentication is enabled on SMB. How many shares is Nest showing on SMB?

<figure><img src="../../../../.gitbook/assets/image (498).png" alt=""><figcaption></figcaption></figure>

#### What is the password of the TempUser?

We find some interesting shares with no password needed ->

<figure><img src="../../../../.gitbook/assets/image (496).png" alt=""><figcaption></figcaption></figure>

Using telnet we get what seems to be a shell of some kind with a service name HQK Reporting ->

<figure><img src="../../../../.gitbook/assets/image (499).png" alt=""><figcaption></figcaption></figure>

we navigate through the environment but unfortunately a lot of the files are out of our reach:

<figure><img src="../../../../.gitbook/assets/image (500).png" alt=""><figcaption></figcaption></figure>

Let's go back to smb enumeration:

```
smbmap -H 10.129.212.135 -u null
```

<figure><img src="../../../../.gitbook/assets/image (501).png" alt=""><figcaption></figcaption></figure>

1. `smbmap`: This is the command to use the `smbmap` tool from the Impacket suite.
2. `-H 10.129.212.135`: This option specifies the IP address of the SMB server to connect to (`10.129.212.135`).
3. `-u null`: This option specifies the username to use when connecting to the SMB server. In this case, the username is set to "null", which can be used to attempt a null session connection to the SMB server. A null session connection is an unauthenticated connection that can be used to enumerate shares and perform other tasks, depending on the SMB server's configuration.

Okay this one blew my mind and i'm so glad to know that now:

<figure><img src="../../../../.gitbook/assets/image (502).png" alt=""><figcaption></figcaption></figure>

* `smb: \> recurse ON`: This command enables the "recurse" option, which allows the `smbclient` tool to traverse the directory structure recursively when performing file operations.
* `smb: \> prompt OFF`: This command disables the "prompt" option, which will prevent the `smbclient` tool from asking for confirmation before executing commands.
* `smb: \> mget *`: This command uses the "mget" command to download all files from the current directory. The wildcard "\*" is used to match all files in the directory.

If we go and read the file we get this interesting email:

```
We would like to extend a warm welcome to our newest member of staff, <FIRSTNAME> <SURNAME>

You will find your home folder in the following location: 
\\HTB-NEST\Users\<USERNAME>

If you have any issues accessing specific services or workstations, please inform the 
IT department and use the credentials below until all systems have been set up for you.

Username: TempUser
Password: welcome2019


Thank you
HR
```

We try to see what changed with those credentials:

<figure><img src="../../../../.gitbook/assets/image (503).png" alt=""><figcaption></figcaption></figure>

we got READONLY on the Secure file

we tried connecting with smbclient but we can't really do anything in the folder:

<figure><img src="../../../../.gitbook/assets/image (504).png" alt=""><figcaption></figcaption></figure>

Go on all the shares and do the recursive trick to mget everything on your computer with the new rights&#x20;

spoiler: the interesting share is the data share with TempUser credentials

then with this command we&#x20;

```
find IT -type f -ls
```

<figure><img src="../../../../.gitbook/assets/image (505).png" alt=""><figcaption></figcaption></figure>

the RU\_config.xml seems a bit weird so let's go look at it:

<figure><img src="../../../../.gitbook/assets/image (506).png" alt=""><figcaption><p>it's encrypted </p></figcaption></figure>

#### Authenticating on SMB as TempUser gives you read access to the share "Secure$". What is the encrypted (and encoded) password for the user c.smith that you uncovered from the files inside that share?

#### What's the name of the Visual Basic project that will help you decrypt the password?
