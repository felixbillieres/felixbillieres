# 🇨🇿 Windows and NIX shells

### Windows

Here are some of the most exploited vulnerabilities of our time:

<table data-header-hidden><thead><tr><th width="175"></th><th></th></tr></thead><tbody><tr><td><strong>Vulnerability</strong></td><td><strong>Description</strong></td></tr><tr><td><code>MS08-067</code></td><td>MS08-067 was a critical patch pushed out to many different Windows revisions due to an SMB flaw. This flaw made it extremely easy to infiltrate a Windows host. It was so efficient that the Conficker worm was using it to infect every vulnerable host it came across. Even Stuxnet took advantage of this vulnerability.</td></tr><tr><td><code>Eternal Blue</code></td><td>MS17-010 is an exploit leaked in the Shadow Brokers dump from the NSA. This exploit was most notably used in the WannaCry ransomware and NotPetya cyber attacks. This attack took advantage of a flaw in the SMB v1 protocol allowing for code execution. EternalBlue is believed to have infected upwards of 200,000 hosts just in 2017 and is still a common way to find access into a vulnerable Windows host.</td></tr><tr><td><code>PrintNightmare</code></td><td>A remote code execution vulnerability in the Windows Print Spooler. With valid credentials for that host or a low privilege shell, you can install a printer, add a driver that runs for you, and grants you system-level access to the host. This vulnerability has been ravaging companies through 2021. 0xdf wrote an awesome post on it <a href="https://0xdf.gitlab.io/2021/07/08/playing-with-printnightmare.html">here</a>.</td></tr><tr><td><code>BlueKeep</code></td><td>CVE 2019-0708 is a vulnerability in Microsoft's RDP protocol that allows for Remote Code Execution. This vulnerability took advantage of a miss-called channel to gain code execution, affecting every Windows revision from Windows 2000 to Server 2008 R2.</td></tr><tr><td><code>Sigred</code></td><td>CVE 2020-1350 utilized a flaw in how DNS reads SIG resource records. It is a bit more complicated than the other exploits on this list, but if done correctly, it will give the attacker Domain Admin privileges since it will affect the domain's DNS server which is commonly the primary Domain Controller.</td></tr><tr><td><code>SeriousSam</code></td><td>CVE 2021-36924 exploits an issue with the way Windows handles permission on the <code>C:\Windows\system32\config</code> folder. Before fixing the issue, non-elevated users have access to the SAM database, among other files. This is not a huge issue since the files can't be accessed while in use by the pc, but this gets dangerous when looking at volume shadow copy backups. These same privilege mistakes exist on the backup files as well, allowing an attacker to read the SAM database, dumping credentials.</td></tr><tr><td><code>Zerologon</code></td><td>CVE 2020-1472 is a critical vulnerability that exploits a cryptographic flaw in Microsoft’s Active Directory Netlogon Remote Protocol (MS-NRPC). It allows users to log on to servers using NT LAN Manager (NTLM) and even send account changes via the protocol. The attack can be a bit complex, but it is trivial to execute since an attacker would have to make around 256 guesses at a computer account password before finding what they need. This can happen in a matter of a few seconds.</td></tr></tbody></table>

To determine if the target is a windows machine, you can look at the ttl that will either be 32 of 128

```shell-session
ElFelixi0@htb[/htb]$ ping 192.168.86.39 

PING 192.168.86.39 (192.168.86.39): 56 data bytes
64 bytes from 192.168.86.39: icmp_seq=0 ttl=128 time=102.920 ms
```

With nmap we can try looking for the OS with the -O flag

```shell-session
ElFelixi0@htb[/htb]$ sudo nmap -v -O 192.168.86.39
```

to grab the banner of the target we can use the following script:

```shell-session
ElFelixi0@htb[/htb]$ sudo nmap -v 192.168.86.39 --script banner.nse
```

***

When it comes to creating payloads for Windows hosts, we can choose many file types ->

[DLLs](https://docs.microsoft.com/en-us/troubleshoot/windows-client/deployment/dynamic-link-library) that are used in microsoft to provide shared code and data that can be used by many different programs at once

[Batch](https://commandwindows.com/batch.htm) files that are text-based DOS scripts utilized by system administrators to complete multiple tasks through the command-line interpreter

[VBS](https://www.guru99.com/introduction-to-vbscript.html) that is a lightweight scripting language based on Microsoft's Visual Basic.

[MSI](https://docs.microsoft.com/en-us/windows/win32/msi/windows-installer-file-extensions) `.MSI` files serve as an installation database for the Windows Installer.

[Powershell](https://docs.microsoft.com/en-us/powershell/scripting/overview?view=powershell-7.1) Powershell is both a shell environment and scripting language.

***

Here is the difference bteween CMD and Powershell ->

Use `CMD` when:

* You are on an older host that may not include PowerShell.
* When you only require simple interactions/access to the host.
* When you plan to use simple batch files, net commands, or MS-DOS native tools.
* When you believe that execution policies may affect your ability to run scripts or other actions on the host.

Use `PowerShell` when:

* You are planning to utilize cmdlets or other custom-built scripts.
* When you wish to interact with .NET objects instead of text output.
* When being stealthy is of lesser concern.
* If you are planning to interact with cloud-based services and hosts.
* If your scripts set and use Aliases.

### Unix/Linux

If we dsicover this webpage during enumeration

<figure><img src="../../../.gitbook/assets/image (1448).png" alt=""><figcaption></figcaption></figure>

We can go and look at&#x20;

<figure><img src="../../../.gitbook/assets/image (1449).png" alt=""><figcaption></figcaption></figure>

or in msfconsole

```shell-session
msf6 > search rconfig
msf6 > use exploit/linux/http/rconfig_vendors_auth_file_upload_rce
```

And if we ever want to spawn a full TTY shell ->

```shell-session
python -c 'import pty; pty.spawn("/bin/sh")' 
```
