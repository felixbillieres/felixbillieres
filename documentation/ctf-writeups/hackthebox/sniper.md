# Sniper

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

First we start by an nmap :

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

I first go on the webpage, and find a nice interface with sub categories:

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

then click on the user interface that seemed the only legit page on the homepage:

decided to create a user Felix/Felix:

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

and when trying to log in:&#x20;

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

So I start to wonder if the other pages weren't fake rabbit holes:

In the "our service" page, we find a dropdown menu for the languages linked to a php file that could lead to RFI

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

If we try and go fetch a file in a subdirectory:

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

```
http://10.129.229.6/blog/?lang=/windows/system32/drivers/etc/hosts
```
