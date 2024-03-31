# 👁️ exam

{% embed url="https://www.thepastamentors.com/" %}

**possible people/usernames ->**

\-Alessandra Fettuccini

\-Alanzo Bucatini

\-Adriano Penne

\-Ferruccio Tortellini and Giovanni Rigatoni (chef in training, new account default password?)

\-**WEB ADMIN:** Leo Fusilli

**Adress of restaurant:**

500 Terry Francois Street San Francisco, CA 94158

**mail:**

info@thepastamentors.com

<figure><img src=".gitbook/assets/image (755).png" alt=""><figcaption></figcaption></figure>

nmap output:

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-30 03:36 EDT
Nmap scan report for 10.10.155.5
Host is up (0.11s latency).
Not shown: 991 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ca:8d:f9:d8:62:2f:b9:df:dd:c2:af:91:9a:7a:c8:18 (RSA)
|   256 74:27:39:90:00:13:ab:60:ce:ae:68:68:77:ff:d2:41 (ECDSA)
|_  256 fe:a4:f4:52:1f:01:62:08:4b:96:2d:49:f4:06:85:cb (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: SMTP: EHLO 521 5.5.1 Protocol error\x0D
80/tcp  open  http     nginx
|_http-title: Did not follow redirect to https://10.10.155.5/
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: RESP-CODES UIDL CAPA SASL PIPELINING AUTH-RESP-CODE STLS TOP
| ssl-cert: Subject: commonName=mail.thepastamentors.com/organizationName=mail.thepastamentors.com/stateOrProvinceName=GuangDong/countryName=CN
| Not valid before: 2021-04-05T20:22:31
|_Not valid after:  2031-04-03T20:22:31
|_ssl-date: TLS randomness does not represent time
143/tcp open  imap     Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=mail.thepastamentors.com/organizationName=mail.thepastamentors.com/stateOrProvinceName=GuangDong/countryName=CN
| Not valid before: 2021-04-05T20:22:31
|_Not valid after:  2031-04-03T20:22:31
|_imap-capabilities: ENABLE more ID STARTTLS IDLE have capabilities IMAP4rev1 listed post-login LOGINDISABLEDA0001 LITERAL+ LOGIN-REFERRALS Pre-login SASL-IR OK
443/tcp open  ssl/http nginx
| tls-alpn: 
|   h2
|_  http/1.1
|_http-title: Site doesn't have a title (text/html).
| http-robots.txt: 1 disallowed entry 
|_/
| ssl-cert: Subject: commonName=mail.thepastamentors.com/organizationName=mail.thepastamentors.com/stateOrProvinceName=GuangDong/countryName=CN
| Not valid before: 2021-04-05T20:22:31
|_Not valid after:  2031-04-03T20:22:31
|_ssl-date: TLS randomness does not represent time
| tls-nextprotoneg: 
|   h2
|_  http/1.1
587/tcp open  smtp     Postfix smtpd
|_smtp-commands: mail.thepastamentors.com, PIPELINING, SIZE 15728640, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8
| ssl-cert: Subject: commonName=mail.thepastamentors.com/organizationName=mail.thepastamentors.com/stateOrProvinceName=GuangDong/countryName=CN
| Not valid before: 2021-04-05T20:22:31
|_Not valid after:  2031-04-03T20:22:31
|_ssl-date: TLS randomness does not represent time
993/tcp open  ssl/imap Dovecot imapd (Ubuntu)
|_imap-capabilities: ENABLE AUTH=LOGINA0001 ID AUTH=PLAIN IDLE more capabilities IMAP4rev1 have post-login listed LITERAL+ LOGIN-REFERRALS Pre-login SASL-IR OK
| ssl-cert: Subject: commonName=mail.thepastamentors.com/organizationName=mail.thepastamentors.com/stateOrProvinceName=GuangDong/countryName=CN
| Not valid before: 2021-04-05T20:22:31
|_Not valid after:  2031-04-03T20:22:31
|_ssl-date: TLS randomness does not represent time
995/tcp open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: RESP-CODES UIDL CAPA SASL(PLAIN LOGIN) USER AUTH-RESP-CODE PIPELINING TOP
| ssl-cert: Subject: commonName=mail.thepastamentors.com/organizationName=mail.thepastamentors.com/stateOrProvinceName=GuangDong/countryName=CN
| Not valid before: 2021-04-05T20:22:31
|_Not valid after:  2031-04-03T20:22:31
|_ssl-date: TLS randomness does not represent time
Service Info: Hosts: -mail.thepastamentors.com,  mail.thepastamentors.com; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.51 seconds
```

going on the ip 10.10.155.5 goes on this website:

<figure><img src=".gitbook/assets/image (756).png" alt=""><figcaption></figcaption></figure>

roundcube has known exploits:

<figure><img src=".gitbook/assets/image (757).png" alt=""><figcaption></figcaption></figure>

after a feroxbuster scan, i was able to get some informations/directories:

```
200      GET       53l      165w     1083c https://10.10.155.5/iredadmin/static/default/css/reset.css
200      GET      914l     6764w    53934c https://10.10.155.5/iredadmin/static/default/css/screen.css
200      GET        2l       11w     1769c https://10.10.155.5/iredadmin/static/favicon.ico
200      GET      119l      230w     5138c https://10.10.155.5/iredadmin/login
200      GET      119l      230w     5138c https://10.10.155.5/iredadmin
200      GET      453l     2034w    16254c https://10.10.155.5/mail/plugins/password/README
200      GET       23l      113w      868c https://10.10.155.5/mail/skins/classic/README
```

there are several login forms:

<figure><img src=".gitbook/assets/image (758).png" alt=""><figcaption><p><a href="https://10.10.155.5/iredadmin/login">https://10.10.155.5/iredadmin/login</a></p></figcaption></figure>

<figure><img src=".gitbook/assets/image (761).png" alt=""><figcaption><p><a href="https://10.10.155.5/iredadmin">https://10.10.155.5/iredadmin</a></p></figcaption></figure>

and i had access to README file:

maybe interesting finding:

<figure><img src=".gitbook/assets/image (762).png" alt=""><figcaption></figcaption></figure>

a bit less interesting:

<figure><img src=".gitbook/assets/image (763).png" alt=""><figcaption></figcaption></figure>

before brutefoce, i tried looking at other ports -> POP server via telnet is possible:

<figure><img src=".gitbook/assets/image (764).png" alt=""><figcaption></figcaption></figure>

i had this error:&#x20;

\-ERR \[AUTH] Plaintext authentication disallowed on non-secure (SSL/TLS) connections.

I tried looking up the osint side of things and looked for the web admin leo fusilli:[https://twitter.com/jlfusilli](https://twitter.com/jlfusilli)

<figure><img src=".gitbook/assets/image (765).png" alt=""><figcaption><p>jlfusilli</p></figcaption></figure>

maybe now we can bruteforce? possible username for the login forms

i tried bruteforcing everything with this username but without success, now i open up msfconsole and look for roundcube webmail:

i find a auxiliary/gather/roundcube\_auth\_file\_read exploit which means i can read files i didnt have access to earlier in my enumeration, possibly usernames and passwords too ->

when we click on the web admin link on the TPM page, we are able to send a mail to: leo@thepastamentors.com

maybe a username is leo

decided to look at all of the employees:

{% embed url="https://www.linkedin.com/in/alessandra-fettuccini/" %}

{% embed url="https://www.linkedin.com/in/alanzo-bucatini-43b4b425b/" %}

<figure><img src=".gitbook/assets/image (766).png" alt=""><figcaption><p><a href="https://www.emailsherlock.com/externalredirects/adriano.penne@mail.thepastamentors.com/full_name"><em>Name:</em> Adriano Penne</a> adriano.penne@mail.thepastamentors.com</p></figcaption></figure>

maybe a username&#x20;

***

while looking for possible exploits for the roundcube webmail login page ->

I found this:

```
Roundcube Webmail 0.3.1 - Cross-Site Request Forgery / SQL Injection            | php/webapps/17957.txt
```

<figure><img src=".gitbook/assets/image (767).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (768).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (769).png" alt=""><figcaption></figcaption></figure>

on the login form we find the rceversion of the roundcube:

<figure><img src=".gitbook/assets/image (770).png" alt=""><figcaption></figcaption></figure>

Now i'll try to bruteforce using the nomenclature of the mail ->

<figure><img src=".gitbook/assets/image (771).png" alt=""><figcaption></figcaption></figure>

so i can create a username db for all the known employees

alessandra@thepastamentors.com

alanzo@thepastamentors.com

adriano@thepastamentors.com

giovanni@thepastamentors.com&#x20;

ferruccio@thepastamentors.com

leo@thepastamentors.com

i'll try with various lists: found on this github:

<figure><img src=".gitbook/assets/image (773).png" alt=""><figcaption><p>only the password related ones</p></figcaption></figure>

<figure><img src=".gitbook/assets/image (772).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (774).png" alt=""><figcaption></figcaption></figure>

after reading roe, tried the following lists:

![](<.gitbook/assets/image (752).png>)

also tried to fuzz with all the users (allessandra, alanzo, adriano, ferruccio, giovanni, leo and info @thepastamentors.com) and all the word lists, no success:

<figure><img src=".gitbook/assets/image (753).png" alt=""><figcaption></figcaption></figure>

in a desperate attempt, i tried pasta word list but didn't expect much

<figure><img src=".gitbook/assets/image (754).png" alt=""><figcaption></figcaption></figure>

looking on internet i found this :

&#x20;

<figure><img src=".gitbook/assets/image (751).png" alt=""><figcaption></figcaption></figure>

the @demo.iredmail.org made me remember the mail.thepastamentors.com

so let's try one last thing:

giovanni@mail.thepastamentors.com

ferruccio@mail.thepastamentors.com
