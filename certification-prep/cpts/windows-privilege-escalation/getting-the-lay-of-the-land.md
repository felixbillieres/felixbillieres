# 🪂 Getting the Lay of the Land

Gathering network information is a crucial part of our enumeration. We cna look for dual-homed hosts to move to an internal network, gather information about the local domain, view the ARP cache for each interface and view other hosts the host has recently communicated with.

check **Interface(s), IP Address(es), DNS Information >**

```cmd-session
C:\htb> ipconfig /all
```

check **ARP Table >**

```cmd-session
C:\htb> arp -a
```

check **Routing Table >**

```cmd-session
C:\htb> route print
```

Next step is to **enumerate protections** on the host that could be monitoring us or blockin our action such as  blocking non-admin users from running `cmd.exe` or `powershell.exe` or other binaries and file types not needed for their day-to-day work. A popular solution offered by Microsoft is [AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview). We can use the [GetAppLockerPolicy](https://docs.microsoft.com/en-us/powershell/module/applocker/get-applockerpolicy?view=windowsserver2019-ps) cmdlet to enumerate the local, effective (enforced), and domain AppLocker policies.

We can start by **Checking Windows Defender Status >**

```powershell-session
PS C:\htb> Get-MpComputerStatus
<SNIP>
AntivirusEnabled                : True
<SNIP>
```

Then to get more informations, we can **List AppLocker Rules**

```powershell-session
PS C:\htb> Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

And finish of with a **Test of AppLocker Policy**

```powershell-session
PS C:\htb> Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone
```

## Initial Enumeration

We can escalate privileges to one of the following depending on the system configuration and what type of data we encounter

|                                                                                                                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The highly privileged `NT AUTHORITY\SYSTEM` account, or [LocalSystem](https://docs.microsoft.com/en-us/windows/win32/services/localsystem-account) account which is a highly privileged account with more privileges than a local administrator account and is used to run most Windows services. |
| The built-in local `administrator` account. Some organizations disable this account, but many do not. It is not uncommon to see this account reused across multiple systems in a client environment.                                                                                              |
| Another local account that is a member of the local `Administrators` group. Any account in this group will have the same privileges as the built-in `administrator` account.                                                                                                                      |
| A standard (non-privileged) domain user who is part of the local `Administrators` group.                                                                                                                                                                                                          |
| A domain admin (highly privileged in the Active Directory environment) that is part of the local `Administrators` group.                                                                                                                                                                          |

When we gain initial shell access to the host, it is vital to gain situational awareness and uncover details relating to the OS version, patch level, installed software, current privileges, group memberships, and more.

### System Information

Looking at the system itself will give us a better idea of the exact operating system version, hardware in use, installed programs, and security updates. This will help us narrow down our hunt for any missing patches and associated CVEs that we may be able to leverage to escalate privileges.

```cmd-session
C:\htb> tasklist /svc
<...SNIP...>
svchost.exe                   2004 AppHostSvc
FileZilla Server.exe          1140 FileZilla Server
inetinfo.exe                  1164 IISADMIN
```

We need to be familiar with Windows processes such as [Session Manager Subsystem (smss.exe)](https://en.wikipedia.org/wiki/Session\_Manager\_Subsystem), [Client Server Runtime Subsystem (csrss.exe)](https://en.wikipedia.org/wiki/Client/Server\_Runtime\_Subsystem), [WinLogon (winlogon.exe)](https://en.wikipedia.org/wiki/Winlogon), [Local Security Authority Subsystem Service (LSASS)](https://en.wikipedia.org/wiki/Local\_Security\_Authority\_Subsystem\_Service), and [Service Host (svchost.exe)](https://en.wikipedia.org/wiki/Svchost.exe)

In this output we would need to be  interested in the `FileZilla` FTP server running and would attempt to enumerate the version to look for public vulnerabilities or misconfigurations such as FTP anonymous access

**Display All Environment Variables**

If the folder placed in the PATH is writable by your user, it may be possible to perform DLL Injections against other applications. Remember, when running a program, Windows looks for that program in the CWD (Current Working Directory) first, then from the PATH going left to right. This means if the custom path is placed on the left (before C:\Windows\System32), it is much more dangerous than on the right.

```cmd-session
C:\htb> set

ALLUSERSPROFILE=C:\ProgramData
APPDATA=C:\Users\Administrator\AppData\Roaming
CommonProgramFiles=C:\Program Files\Common Files
<SNIP>
```

**View Detailed Configuration Information**

The `systeminfo` command will show if the box has been patched recently and if it is a VM. If the box has not been patched recently, getting administrator-level access may be as simple as running a known exploit.  The `System Boot Time` and `OS Version` can also be checked to get an idea of the patch level

```cmd-session
C:\htb> systeminfo
<SNIP>
OS Name:                   Microsoft Windows Server 2016 Standard
OS Version:                10.0.14393 N/A Build 14393
<SNIP>
BIOS Version:              VMware, Inc. VMW71.00V.16707776.B64.2008070230, 8/7/2020
<SNIP>
Network Card(s):           2 NIC(s) Installed.
                                 [01]: 10.129.43.8
<SNIP>                                                            
                                  [01]: 192.168.20.56
```

If `systeminfo` doesn't display hotfixes, they may be queriable with [WMI](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page) using the WMI-Command binary with [QFE](https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/win32-quickfixengineering)

```cmd-session
C:\htb> wmic qfe
```

We can do this with PowerShell as well

```powershell-session
PS C:\htb> Get-HotFix | ft -AutoSize
```

**Installed Programs**

WMI can also be used to display installed software and after that  Run `LaZagne` to check if stored credentials for those applications are installed

```cmd-session
C:\htb> wmic product get name
```

with PowerShell:

```powershell-session
PS C:\htb> Get-WmiObject -Class Win32_Product |  select Name, Version
```

**Display Running Processes**

The [netstat](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/netstat) command will display active TCP and UDP connections which will give us a better idea of what services are listening on which port(s) both locally and accessible to the outside.

```cmd-session
PS C:\htb> netstat -ano
```

### User & Group Information

**Logged-In Users**

It is always important to determine what users are logged into a system

```cmd-session
C:\htb> query user
<SNIP>
>administrator         rdp-tcp#2           1  Active          .  3/25/2021 9:27 AM
```

**Current User**

We always need to check what user context our account is running under first. Suppose we gain access as a service account. In that case, we may have privileges such as `SeImpersonatePrivilege`, which can often be easily abused to escalate privileges using a tool such as [Juicy Potato](https://github.com/ohpe/juicy-potato).

```cmd-session
C:\htb> echo %USERNAME%
```

**Current User Privileges**

```cmd-session
C:\htb> whoami /priv
```

**Current User Group Information**

important to check if our user inherited any rights through their group membership? Are they privileged in the Active Directory domain environment

```cmd-session
C:\htb> whoami /groups
```

**Get All Users**

Knowing what other users are on the system is important as well. If we captured for a user `bob`, and see a `bob_adm` user in the local administrators group, it is worth checking for credential re-use

```cmd-session
C:\htb> net user
```

**Get All Groups**

```cmd-session
C:\htb> net localgroup
```

**Details About a Group**

we may find a password or other interesting information stored in the group's description

```cmd-session
C:\htb> net localgroup administrators
```

**Get Password Policy & Other Account Information**

```cmd-session
C:\htb> net accounts
```

_**What service is listening on port 8080 (service name not the executable)?**_

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## Communication with Processes
