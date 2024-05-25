# 🎟️ Golden Ticket Attacks

<figure><img src="../../../../.gitbook/assets/image (633).png" alt=""><figcaption></figcaption></figure>

It is called a "golden ticket" because it provides the attacker with unrestricted access to the domain, similar to a VIP pass or a "golden ticket" to a concert or event.

A golden ticket is created by using the Kerberos Authentication Service (AS) REP and Ticket Granting Service (TGS) REP tickets to create a forged Kerberos Ticket-Granting Ticket (TGT). The TGT is then used to request additional service tickets, allowing the attacker to access any resource in the domain without authentication.

To attack golden tickets we'll use mimikatz:

<figure><img src="../../../../.gitbook/assets/image (634).png" alt=""><figcaption></figcaption></figure>

```
lsadump::lsa /inject /name:krbtgt
```

The command `lsadump::lsa /inject /name:krbtgt` is used to extract the Kerberos Ticket-Granting Ticket (TGT) for the Krbtgt account from the LSASS (Local Security Authority Subsystem Service) process on a Windows system.

The `/inject` option specifies that the command should inject a DLL (Dynamic Link Library) into the LSASS process to extract the T. This is a more advanced technique than simply reading the TGT from memory

The `/name:krbtgt` option specifies that the command should extract TGT for the Krbtgt account. The Krbtgt account is a highly privileged account used by the Kerberos authentication protocol to issue TGTs for users and services in the domain. By extracting the TGT for the Krbtgt account, the attacker can create a golden ticket and use it to gain unauthorized access to domain.

<figure><img src="../../../../.gitbook/assets/image (636).png" alt=""><figcaption></figcaption></figure>

To make this attack succesfull, we need the krbtgt hash and the domain SID

once those are known

```
kerberos::golden /User:Administrator /domain:marvel.local /sid:[SID NUMBER] /id:500
```

The SID for the KRBTGT account is S-1-5--502 and lives in the Users OU in the domain by default. Microsoft does not recommend moving this account to another OU.

{% embed url="https://tools.thehacker.recipes/mimikatz/modules/kerberos/golden" %}

<figure><img src="../../../../.gitbook/assets/image (635).png" alt=""><figcaption><p>that's an SID</p></figcaption></figure>

{% embed url="https://book.hacktricks.xyz/v/fr/windows-hardening/active-directory-methodology/golden-ticket" %}

{% embed url="https://adsecurity.org/?p=483" %}

Once we gain a shell we can do any command on any user in the domain ->

```
misc::cmd
```

To open the cmd where we can utilize the session and the golden ticket.

Now we have access all over the Domain.

<figure><img src="../../../../.gitbook/assets/image (637).png" alt=""><figcaption></figcaption></figure>
