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

<figure><img src="../../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Once we control a user, we can check what remote access rights they have either directly or inherited via group membership under `Execution Rights` on the `Node Info` tab.

<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption><p>or <code>Analysis</code> tab and run the pre-built queries <code>Find Workstations where Domain Users can RDP</code> or <code>Find Servers where Domain Users can RDP</code>.</p></figcaption></figure>

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

<figure><img src="../../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

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
