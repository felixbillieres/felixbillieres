# 🛳️ Pass Attacks

## Comprehensive Explanation on Password Attacks and Lateral Movement

### Introduction

Password attacks are techniques cyber adversaries use to gain unauthorized access and control over computer systems. Understanding these concepts is crucial for defending against potential threats.

<figure><img src="../../../../.gitbook/assets/image (194).png" alt=""><figcaption></figcaption></figure>

### Password Dumping

When attackers successfully retrieve passwords, either through techniques like hash dumping or extracting clear text passwords, they gain a significant advantage. These stolen credentials can be leveraged for lateral movement within a network.

### Utilizing CrackMapExec for SMB Pass-around

CrackMapExec is a powerful tool that facilitates the passing of credentials around the network, particularly over the Server Message Block (SMB) protocol. This is a common method attackers use to propagate across networks.

### Hashdump Techniques

Hashdumping involves extracting password hashes from compromised systems. Metasploit and similar tools allow attackers to run hashdump commands, revealing hashed passwords for further exploitation.

### Secrets Dumping

Secrets dumping, often performed with tools like Impacket's secretsdump, goes beyond hashdumping. It extracts not only hashes but also provides a comprehensive view of all accounts associated with those hashes, paving the way for potential attacks.

### Pass-the-Hash Attack

The culmination of these activities leads to a "Pass-the-Hash" attack. Attackers use stolen hashed credentials to authenticate and move laterally within the network, escalating privileges along the way.

### SAM and LSA Dumping

CrackMapExec enables the dumping of Security Account Manager (SAM) on every device. SAM contains local account information, and attackers can exploit this data for unauthorized access.

### Enumerating Shares

Another tactic involves enumerating shares on devices, revealing what resources are available. This information aids attackers in identifying potential targets for exploitation.

### LSA Dump for Juicy Details

Dumping the Local Security Authority (LSA) provides attackers with valuable details. Modules like Lsassy in CrackMapExec can extract and store credentials from the LSASS process.

### CrackMapExec Database

CrackMapExec maintains a database that stores information about attempted attacks. This database aids attackers in organizing and referencing their efforts for future actions.

Understanding these concepts is crucial for both cybersecurity professionals and beginners to fortify networks against potential threats.

### 🌐 Sources

1. [thehacker.recipes - Pass the hash](https://www.thehacker.recipes/ad/movement/ntlm/pth)
2. [whiteoaksecurity.com - Dumping LSASS W/ No Mimikatz](https://www.whiteoaksecurity.com/blog/attacks-defenses-dumping-lsass-no-mimikatz/)
3. [notes.qazeer.io - Credentials dumping](https://notes.qazeer.io/windows/post\_exploitation/credentials\_dumping)
4. [haax.fr - CrackMapExec](https://cheatsheet.haax.fr/windows-systems/exploitation/crackmapexec/)
5. [hacktricks.xyz - Stealing Windows Credentials](https://book.hacktricks.xyz/windows-hardening/stealing-credentials)
6. [thehacker.recipes - SAM & LSA secrets](https://www.thehacker.recipes/ad/movement/credentials/dumping/sam-and-lsa-secrets)
