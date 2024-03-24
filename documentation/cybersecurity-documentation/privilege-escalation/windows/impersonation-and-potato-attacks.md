# 🥔 Impersonation and Potato Attacks

I**mpersonation**: This is a legitimate technique in Windows programming that allows a thread or process to execute with the security context of a different user or security principal. Impersonation is commonly used in scenarios where a service or application needs to access resources on behalf of a user, without requiring the user's credentials.&#x20;

However, in the context of privilege escalation, attackers can abuse impersonation to escalate their privileges. By impersonating a higher-privileged user, such as an administrator, attackers can gain access to resources or perform actions that would otherwise be restricted. This is often achieved by exploiting vulnerabilities in applications or services running with elevated privileges.

***

**Potato Attacks**: This refers to a class of privilege escalation techniques that exploit the Windows security model to elevate privileges. The term "Potato" originates from the "token kidnapping" technique, where attackers hijack a privileged token to gain elevated access.

One well-known example of a Potato Attack is the "Print Spooler Service" exploit, also known as "PrintNightmare." By abusing the Print Spooler service, attackers can execute arbitrary code with SYSTEM privileges, effectively gaining full control over the system.

Potato Attacks typically involve manipulating Windows services or components that run with elevated privileges, exploiting misconfigurations or vulnerabilities to execute arbitrary code with higher privileges than the attacker initially possessed.

<figure><img src="../../../../.gitbook/assets/image (27) (1).png" alt=""><figcaption></figcaption></figure>

***

### Token Impersonation <a href="#lecture_heading" id="lecture_heading"></a>

token =cookies for your computer, allows access to system/network without credentials

there is delegate tokens and impersonate tokens&#x20;

<figure><img src="../../../../.gitbook/assets/image (28) (1).png" alt=""><figcaption></figcaption></figure>

impersonate token in meterpreter&#x20;

```
list_tokens -u 
```

using meterpreter, you can also use this syntax:

```css
meterpreter > impersonate_token [SID]
```

After compromising a system and gaining access with Meterpreter, you can use built-in commands to list users and their SIDs. For example:

```
meterpreter > getuid
meterpreter > getsystem
meterpreter > list_users
```

SIDs are stored in the Windows Registry, and you can access them directly using Meterpreter or other tools. For example:

```
meterpreter > reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ProfileList
```

If the target system is part of an Active Directory domain, you can use tools like `net`, `dsquery`, or `PowerShell` to query Active Directory for user information including SIDs. For example:

```bash
net user username /domain
```

PowerShell provides cmdlets to retrieve user information, including SIDs. For example:

```powershell
Get-LocalUser
```

If we're in an AD, we can find some hashes which could mean domain compromising:

```
Invoke-Mimikatz -Command '"privilege::debug" "LSADump::LSA /patch" exit' -Computer [example-Domain][HYDRA.marvel.local]
```

1. **Invoke-Mimikatz**: This is a PowerShell command that executes the Mimikatz tool within the context of PowerShell. Invoke-Mimikatz simplifies the usage of Mimikatz by providing a PowerShell interface to execute Mimikatz commands.
2. **-Command**: This parameter specifies the Mimikatz commands to execute. In this case, the command string is enclosed within double quotes.
3. **"privilege::debug"**: This is the first Mimikatz command being executed. It enables debug privileges, which allows the process running Mimikatz to perform actions that require higher privileges than it currently possesses. Debug privileges are often necessary for accessing certain parts of the system's memory.
4. **"LSADump::LSA /patch"**: This is the second Mimikatz command being executed. It instructs Mimikatz to perform LSA (Local Security Authority) dump, which extracts credential information from memory related to the Local Security Authority Subsystem Service (LSASS). The "/patch" option specifies that Mimikatz should attempt to patch LSASS memory to prevent detection by certain security tools or techniques.
5. **exit**: This command tells Mimikatz to exit after executing the previous commands. This ensures that Mimikatz doesn't remain running indefinitely after completing its tasks.

Then once, it runs, throw in a&#x20;

```
privilege::debug
#following a 
LSADump::LSA /patch
```

{% embed url="https://book.hacktricks.xyz/windows-hardening/stealing-credentials/credentials-mimikatz" %}

<figure><img src="../../../../.gitbook/assets/image (30) (1).png" alt=""><figcaption><p>very sensitive hashes to create golden tickets</p></figcaption></figure>

{% embed url="https://blog.3or.de/mimikatz-deep-dive-on-lsadumplsa-patch-and-inject" %}

### Impersonation Privileges <a href="#lecture_heading" id="lecture_heading"></a>

If we owned a machine, we can run the&#x20;

```
whoami /priv
```

<figure><img src="../../../../.gitbook/assets/image (31) (1).png" alt=""><figcaption></figcaption></figure>

there are some bad privileges that would make it possible for us to use impersonation like&#x20;

**SeImpersonatePrivilege :** [**https://learn.microsoft.com/fr-fr/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege**](https://learn.microsoft.com/fr-fr/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege)

but there are many more and when doing this always check for a privesc opportunity

there is also the meterpreter version of **whoami /priv**

```
getprivs
```

<figure><img src="../../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

See which ones are potentially good for us:

{% embed url="https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/windows-privilege-escalation/#eop-impersonation-privileges" %}

### Potato Attacks <a href="#lecture_heading" id="lecture_heading"></a>

The nerd part about potato attacks, very good doc ->

{% embed url="https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/" %}

> The idea behind this vulnerability is simple to describe at a high level:
>
> 1. Trick the “NT AUTHORITY\SYSTEM” account into authenticating via NTLM to a TCP endpoint we control.
> 2. Man-in-the-middle this authentication attempt (NTLM relay) to locally negotiate a security token for the “NT AUTHORITY\SYSTEM” account. This is done through a series of Windows API calls.
> 3. Impersonate the token we have just negotiated. This can only be done if the attackers current account has the privilege to impersonate security tokens. This is usually true of most service accounts and not true of most user-level accounts.

another version of rotten potato called juicy potato : [https://github.com/ohpe/juicy-potato](https://github.com/ohpe/juicy-potato)

To use when you have `SeImpersonate` or `SeAssignPrimaryToken` privileges

Difference between the 2:

1. **Rotten Potato**:
   * **Technique**: Rotten Potato, also known as "RogueWinRM," leverages Windows Management Instrumentation (WMI) and NTLM relay attacks to escalate privileges on a compromised system.
   * **Method**: It involves setting up a rogue HTTP server that listens for NTLM authentication requests. When a user with administrative privileges authenticates to a remote Windows Management Instrumentation service (WinRM), Rotten Potato relays the authentication request to the local NTLM authentication service (LSASS) using a technique called NTLM relay.
   * **Objective**: The objective is to trick the target system into authenticating the attacker, thereby granting them administrative privileges.
2. **Juicy Potato**:
   * **Technique**: Juicy Potato, also known as "CLSID spoofing," exploits the Windows Component Object Model (COM) to execute a process with elevated privileges.
   * **Method**: It involves manipulating the registry to create a fake COM object with a CLSID (Class Identifier) pointing to a desired executable. When certain Windows processes (such as DCOM) attempt to instantiate this fake COM object, the specified executable is executed with elevated privileges due to how COM objects are instantiated.
   * **Objective**: Similar to Rotten Potato, the objective is to execute arbitrary code with higher privileges, typically achieving elevated privileges to the SYSTEM level.

### getsystem

What happens when I type getsystem? - [https://blog.cobaltstrike.com/2014/04/02/what-happens-when-i-type-getsystem/](https://blog.cobaltstrike.com/2014/04/02/what-happens-when-i-type-getsystem/)

In Metasploit, the `getsystem` command is used to attempt to escalate privileges on a compromised system. It is commonly used after initial access has been achieved to escalate privileges from the current user to a higher privileged user, typically to the SYSTEM account on Windows systems.
