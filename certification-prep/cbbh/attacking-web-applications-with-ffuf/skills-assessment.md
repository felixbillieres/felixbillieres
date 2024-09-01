# Skills Assessment

_**Run a sub-domain/vhost fuzzing scan on '\*.academy.htb' for the IP shown above. What are all the sub-domains you can identify? (Only write the sub-domain name)**_

```
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:55350 -H "Host: FUZZ.academy.htb" -fs 985
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**Before you run your page fuzzing scan, you should first run an extension fuzzing scan. What are the different extensions accepted by the domains?**_

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://faculty.academy.htb:55350/indexFUZZps
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**One of the pages you will identify should say 'You don't have access!'. What is the full page URL?**_



_**In the page from the previous question, you should be able to find multiple parameters that are accepted by the page. What are they?**_



_**Try fuzzing the parameters you identified for working values. One of them should return a flag. What is the content of the flag?**_
