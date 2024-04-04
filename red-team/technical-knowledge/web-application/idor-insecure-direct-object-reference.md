# 😎 IDOR - Insecure Direct Object Reference

Insecure Direct Object Reference (IDOR) is a security vulnerability that occurs when an application provides unauthorized access to objects (such as files, directories, database records, or URLs) by exposing direct references to them. In other words, an attacker can manipulate input parameters, such as URLs or form fields, to access objects or data they are not supposed to.

<figure><img src="../../../../.gitbook/assets/image (369).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (370).png" alt=""><figcaption></figcaption></figure>

We have a website with a user id that lets us see the user concerned if we change it manualy

<figure><img src="../../../../.gitbook/assets/image (371).png" alt=""><figcaption></figcaption></figure>

Let's try to enumerate all the active user on the server ->

in a number.txt file, copy paste numbers from 1 to 2000 and then fuzz the url:

```
ffuf -u 'http://localhost/labs/e0x02.php?account=FUZZ' -w num2.txt 
```

<figure><img src="../../../../.gitbook/assets/image (372).png" alt=""><figcaption></figcaption></figure>

so we can easily guess those are false positives if we try them on the website we get a "user does not exist" so let's filter it out

```
ffuf -u 'http://localhost/labs/e0x02.php?account=FUZZ' -w num2.txt  -fs 849
```

<figure><img src="../../../../.gitbook/assets/image (373).png" alt=""><figcaption></figcaption></figure>

And here we go we got all the users that exist on the server :tada:
