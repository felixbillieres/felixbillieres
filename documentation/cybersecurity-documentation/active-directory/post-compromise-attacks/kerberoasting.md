# 🐩 Kerberoasting

Kerberoasting is a type of attack used in the context of Microsoft's Kerberos authentication protocol. In this attack, an attacker extracts the Ticket Granting Service (TGS) ticket for a service account from the Kerberos Authentication Service (AS) ticket, and then cracks the password offline using a tool like `tgsrepcrack` or `kerberoast`

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

after you have gained initial access to a target network and have escal your privileges to a domain user account At this point, you can start looking for service accounts that may be vulnerable to Kerberoasting

```
GetUserSPNs.py <Domain/Username:password> -dc-ip <ip of DC> -request
```

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

After Dumping the Hashes, we will use Hashcat for cracking the hashes.

```
hashcat -m 13100 hash.txt wordlist.txt
```

<figure><img src="../../../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>
