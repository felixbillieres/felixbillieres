# 🥸 Pass-the-Hash

Pass the hash (PtH) is a type of attack used in the context of Windows authentication. In this attack, an attacker captures the NTLM hash of a user's password and uses it to authenticate to a remote system without ever cracking or learning the plaintext password. This is possible because Windows systems use NTLM hashes for authentication in many situations, even when the original password is a plaintext string.

<figure><img src="../../../../.gitbook/assets/image (24) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We are going to start with crackmapexec:

```
crackmapexec smb 192.168.138.0/24 -u fcastle -d MARVEL.local -p Password1
```

* `smb`: This specifies the protocol to use for the scan. In this case, SMB (Server Message Block) is chosen, which is a common protocol used for sharing files, printers, and serial ports in a network.
* `192.168.138.0/24`: This is the IP range to scan. In this case, it's scanning the subnet `192.168.138.0/24`, which includes all IP addresses from `192.168.138.1` to `192.168.138.254`.

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1)  (23).png" alt=""><figcaption></figcaption></figure>

Now another variation:

```
crackmapexec smb 192.168.138.0/24 -u administrator -H [NTLM1 hash] --local-auth
```

\-H \[NTLM1 hash]`: This specifies the NTLM hash of the password to use for the scan. In this case, the hash is provided in the command itself, enclosed in square brackets. 6.` --local-auth\`: This option tells CrackMapExec to use the credentials provided in the command to authenticate locally on each target machine before attempting to connect shares

you can add a --shares to enumerate shares for the users

or see plenty of other commands with:

```
crackmapexec smb -L
```

{% content-ref url="../../../../interacting-with-protocols-and-tools/tools/crackmapexec.md" %}
[crackmapexec.md](../../../../interacting-with-protocols-and-tools/tools/crackmapexec.md)
{% endcontent-ref %}

```
cmedb
```

`cmedb` is a database tool used by CrackMapExec (CME) to store and manage data related to penetration testing and post-exploitation activities.

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1)   (2).png" alt=""><figcaption></figcaption></figure>
