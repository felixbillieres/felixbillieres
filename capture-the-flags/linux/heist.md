# 🦹 Heist

<figure><img src="../../../.gitbook/assets/image (465).png" alt=""><figcaption></figcaption></figure>

First we're redirected to this page:

<figure><img src="../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

it's a login page but there is login as guest button that leads us to this page. Let's click on attachments&#x20;

<figure><img src="../../../.gitbook/assets/image (467).png" alt=""><figcaption></figcaption></figure>

If we download the attachment we go on this page

<figure><img src="../../../.gitbook/assets/image (468).png" alt=""><figcaption></figcaption></figure>

thanks to hash type, i see the pattern "$1$" and see that its probably:

<figure><img src="../../../.gitbook/assets/image (470).png" alt=""><figcaption></figcaption></figure>

So i can try to use hash cat with mode = 500 or john this way ->

<figure><img src="../../../.gitbook/assets/image (469).png" alt=""><figcaption></figcaption></figure>

we find this "stealthagent" result

and after a quick google search with an online decoder we found that the other type of hash is  a Cisco Type 7 Password

that we decode online to find this password:

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption><p>Q4)sJu\Y8qz*A3?d</p></figcaption></figure>

after getting all those credentials, we could try to enumerate SMB shares like this:

```
smbclient --list //heist.htb/ -U 'hazard'
```

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

but noting too interesting, so then a possible way to go is to use lookupsid.py from impacket to enumerate users:

```
lookupsid.py hazard:stealth1agent@heist.htb
```

The `lookupsid.py` script is specifically designed to query the Security Identifier (SID) of a given user or group in a Windows Active Directory environment. SIDs are unique identifiers assigned to each security principal (user, group, computer) in Windows domains. By querying SIDs, security analysts and penetration testers can gather valuable information about users and groups within the domain, aiding in privilege escalation, lateral movement, and other attack vectors.

<figure><img src="../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

after a bit of fooling around, let's use evil-winrm to connect via chase:

```
evil-winrm -i heist.htb -u chase -p 'Q4)sJu\Y8qz*A3?d'
```

<figure><img src="../../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

then go and pick up the user flag:

<figure><img src="../../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

after a bit of enumeration we go and see this path:

_**C:\Users\Chase\AppData\Roaming\\**_

* **Application Data**: Many applications store user-specific settings, configuration files, and other data in this directory. This can include things like preferences, customization options, cached data, temporary files, and more.
* **Roaming**: The "Roaming" part of the directory name indicates that the data stored here is designed to "roam" with the user. This means that if a user logs into different computers within a network (e.g., in a corporate environment or with roaming user profiles), their application settings and data stored in the Roaming folder will follow them. This is particularly useful in environments where users might need consistent settings and preferences across multiple machines.

with the "ps" command we see Firefox running:

<figure><img src="../../../.gitbook/assets/image (565).png" alt=""><figcaption></figcaption></figure>



The `ps` command in Linux is used to list the currently running processes on the system. It provides information about the processes that are running in the current session or in the entire system depending on the options used.

1. `ps`: By itself, the `ps` command lists the processes running in the current shell session.
2. `ps -e` or `ps aux`: Lists all processes currently running on the system, including those without a controlling terminal.
3. `ps -f` or `ps -l`: Provides a full-format listing which includes additional details such as the user who owns the process, CPU and memory usage, start time, command name, and more.
4. `ps -u username`: Lists processes owned by a specific user.
5. `ps -p PID`: Shows information about a specific process specified by its Process ID (PID).
6. `ps -e --forest`: Displays the process tree, showing the parent-child relationships between processes.

next step is to download procdump:

{% embed url="https://www.techspot.com/downloads/6550-microsoft-procdump.html" %}

download it on your local machine and use the upload command by evilwinrm to upload it to your session:

```
upload Procdump/procdump64.exe
```

<figure><img src="../../../.gitbook/assets/image (566).png" alt=""><figcaption></figcaption></figure>

* **procdump64.exe**:
  * **Purpose**: Procdump is a utility developed by Sysinternals (now owned by Microsoft) that is used for creating crash dump files of processes based on various triggers.
  * **Usage**: Procdump allows you to monitor a process and automatically create a crash dump when certain conditions are met, such as when the process crashes, hangs, or exceeds CPU usage thresholds.
  * **Example**: You might use procdump to capture a memory dump of a process when it encounters an unhandled exception, allowing for post-mortem analysis to diagnose the cause of the crash.

to use it, take the ID of one of the processes and put in an argument:&#x20;

<figure><img src="../../../.gitbook/assets/image (568).png" alt=""><figcaption></figcaption></figure>

```
.\procdump64.exe -accepteula -ma 6428
```

* `-accepteula`: This option is used to accept the End-User License Agreement (EULA) for Procdump. By including this option, the command acknowledges and accepts the terms of the license agreement without prompting the user.
* `-ma`: This option specifies the type of dump to create. In this case, `-ma` stands for "Full Memory Dump." This option instructs Procdump to create a complete memory dump of the specified process.
* `6428`: This is the Process ID (PID) of the target process. The command tells Procdump to monitor and create a memory dump of the process with the PID 6428.

<figure><img src="../../../.gitbook/assets/image (569).png" alt=""><figcaption></figcaption></figure>

then download the strings file and upload it:

{% embed url="https://learn.microsoft.com/en-us/sysinternals/downloads/strings" %}

<figure><img src="../../../.gitbook/assets/image (567).png" alt=""><figcaption></figcaption></figure>

* **strings64.exe**:
  * **Purpose**: Strings is a command-line utility used to extract readable strings from binary files. It is particularly useful for extracting human-readable text from executables, libraries, or other binary files.
  * **Usage**: Strings searches for sequences of printable characters within a binary file and prints them to the standard output. This can be helpful for analyzing malware, reverse engineering, or understanding the content of compiled code.
  * **Example**: You might use strings to analyze a suspicious executable file to see if it contains any plaintext strings that could reveal information about its purpose or behavior.

And use strings to convert the dmp file to a txt file:

The goal of using the `strings` command on a `.dmp` (memory dump) file is to extract human-readable strings from the binary data contained within the dump. Memory dump files often contain a wide range of information, including text strings such as error messages, filenames, registry keys, and potentially even sensitive information like passwords or encryption keys.

```
cmd /c "strings64.exe -accepteula firefox.exe_240315_191748.dmp > dumpfirefox.txt"
```

<figure><img src="../../../.gitbook/assets/image (570).png" alt=""><figcaption></figcaption></figure>

1. **`cmd /c`**: This part of the command is calling the Windows Command Prompt (`cmd.exe`) and using the `/c` option, which tells it to execute the command that follows and then terminate. It's essentially used to run a single command within the Command Prompt.
2. **`"strings64.exe -accepteula firefox.exe_240315_191748.dmp > dumpfirefox.txt"`**: This is the command that will be executed within the Command Prompt. Let's break it down further:
   * **`strings64.exe`**: This is the executable for the `strings` command designed for Windows. It's used to extract readable strings from binary files.
   * **`-accepteula`**: This option is used to accept the End-User License Agreement (EULA) for the `strings` tool without prompting the user.
   * **`firefox.exe_240315_191748.dmp`**: This is the name of the input file. It's assumed to be a memory dump file (`firefox.exe_240315_191748.dmp` in this case), likely created by a process named Firefox.
   * **`> dumpfirefox.txt`**: This part of the command is using the output redirection operator (`>`) to redirect the output of the `strings64.exe` command to a text file named `dumpfirefox.txt`. This means that the output generated by the `strings64.exe` command (i.e., the extracted strings from the memory dump) will be saved into the `dumpfirefox.txt` file.

On the write-up the command is&#x20;

`findstr "password" ./`**`dumpfirefox.txt`**

but it did not work for me, I used the `MOZ_CRASHREPORTER_RESTART_ARG_1` parameter ->

```
findstr "MOZ_CRASHREPORTER_RESTART_ARG_1" "C:/Windows/System32/spool/drivers/color/dumpfirefox.txt"
```

When you execute this command, `findstr` will search through the contents of the `dumpfirefox.txt` file for any lines that contain the text `"MOZ_CRASHREPORTER_RESTART_ARG_1"`. If it finds any matches, it will print those lines to the console.

<figure><img src="../../../.gitbook/assets/image (571).png" alt=""><figcaption><p>4dD!5}x/re8]FBuZ</p></figcaption></figure>

You can also play around with the args like using "password="

```
findstr "password=" "C:/Windows/System32/spool/drivers/color/dumpfirefox.txt"
```

<figure><img src="../../../.gitbook/assets/image (572).png" alt=""><figcaption><p>4dD!5}x/re8]FBuZ</p></figcaption></figure>

and now on a local session ->

```
evil-winrm -i heist.htb -u administrator -p '4dD!5}x/re8]FBuZ'
```

<figure><img src="../../../.gitbook/assets/image (573).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (574).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (575).png" alt=""><figcaption></figcaption></figure>

