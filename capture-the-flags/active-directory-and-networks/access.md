---
description: https://app.hackthebox.com/machines/156
---

# 🔑 Access

<figure><img src="../../../.gitbook/assets/image (601).png" alt=""><figcaption></figcaption></figure>

after a quick nmap we just find a webserver and ftp \&telnet port open:

<figure><img src="../../../.gitbook/assets/image (602).png" alt=""><figcaption></figcaption></figure>

the webserver seems useless for now

anonymous login is enabled

<figure><img src="../../../.gitbook/assets/image (603).png" alt=""><figcaption></figcaption></figure>

type in the following command:

```
binary
```

In FTP (File Transfer Protocol), the `binary` command is used to set the transfer mode to binary or image mode. This mode is used when transferring files that are not plain text files, such as images, executables, or compressed files.

then cd into all the folders and get the files (Access Control.zip & backup.mdb)

<figure><img src="../../../.gitbook/assets/image (604).png" alt=""><figcaption></figcaption></figure>

for the mdb file, you can utilize the following command:

```
mdb-sql backup.mdb 
```

and for the other tool who is a pst file:

```
readpst filename.pst
```

there are plenty of ways to open it in a GUI, but there is also [https://www.mdbopener.com/](https://www.mdbopener.com/) for easy access:

<figure><img src="../../../.gitbook/assets/image (605).png" alt=""><figcaption></figcaption></figure>

after looking around for a bit we find credentials:

<figure><img src="../../../.gitbook/assets/image (606).png" alt=""><figcaption></figcaption></figure>

The other file had a password, now we can see the content:

<figure><img src="../../../.gitbook/assets/image (607).png" alt=""><figcaption></figcaption></figure>

using the `readpst` command, we generate a mbox file that looks like an email:

<figure><img src="../../../.gitbook/assets/image (608).png" alt=""><figcaption><p>4Cc3ssC0ntr0ller</p></figcaption></figure>

While reading the email, we see clear text credentials

now connect with those credentials via telnet:

<figure><img src="../../../.gitbook/assets/image (609).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (610).png" alt=""><figcaption></figcaption></figure>

Next is privesc:

```
cmdkey /list
```

{% embed url="https://juggernaut-sec.com/runas/" %}

<figure><img src="../../../.gitbook/assets/image (611).png" alt=""><figcaption></figcaption></figure>

the output indicates that there is a stored credential for an interactive logon session to the `ACCESS` domain with the `Administrator` user account.

now we utilize the runas command:

```
C:\Windows\System32\runas.exe /user:ACCESS\Administrator /savecred "C:\Windows\System32\cmd.exe /c TYPE C:\Users\Administrator\Desktop\root.txt > C:\Users\security\root.txt
```

1. **`C:\Windows\System32\runas.exe /user:ACCESS\Administrator /savecred`**:
   * This part of the command invokes the `runas.exe` utility, which is used to run a program with different credentials.
   * `/user:ACCESS\Administrator` specifies the username (`Administrator`) and domain (`ACCESS`) under which the subsequent command will be executed.
   * `/savecred` flag instructs `runas.exe` to save the entered credentials (password) for future use, enabling the command to be executed without requiring manual authentication in the future. This flag essentially stores the credentials in the Windows Credential Manager, allowing for automatic authentication in subsequent executions.
2. **`"C:\Windows\System32\cmd.exe /c TYPE C:\Users\Administrator\Desktop\root.txt > C:\Users\security\root.txt"`**:
   * This part of the command specifies the command to be executed with elevated privileges using `runas.exe`.
   * `"C:\Windows\System32\cmd.exe` launches the Windows Command Prompt (`cmd.exe`).
   * `/c` flag indicates that the subsequent string should be treated as a command to be executed by `cmd.exe`, and then the command specified after `/c` is executed.
   * `TYPE C:\Users\Administrator\Desktop\root.txt` is the command to display the contents of the file `root.txt` located on the desktop of the `Administrator` user.
   * `>` is the output redirection operator, which redirects the output of the `TYPE` command (the contents of `root.txt`) to a file specified after `>`.
   * `C:\Users\security\root.txt` is the path where the output of `TYPE` command (`root.txt`) will be saved. It will be saved in the `root.txt` file in the `security` user's directory.

<figure><img src="../../../.gitbook/assets/image (612).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (613).png" alt=""><figcaption></figcaption></figure>
