# 😵‍💫 Authentication vulnerabilities

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

There are 3 main type of authentication:

* Something you **know**, such as a password or the answer to a security question. These are sometimes called "knowledge factors".
* Something you **have**, This is a physical object such as a mobile phone or security token. These are sometimes called "possession factors".
* Something you **are** or do. For example, your biometrics or patterns of behavior. These are sometimes called "inherence factors".

It's crucial to know the difference between **authentication and authorization**

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Lab: Username enumeration via different responses

We first start by taking the given list and triggering all the usernames so we can bypass the cluster bomb that will take very long without the pro version:&#x20;

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We get user ads with a response way different from the rest, let's go on that road ->

We then trigger the list of password with username ads and see that the good password gives us a 302 response →

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Lab: Username enumeration via subtly different responses

<figure><img src="../../.gitbook/assets/image (11) (2).png" alt=""><figcaption></figcaption></figure>

We first start by looking at the response, adding the username and launching the sniper attack with the username list.

In the intruder output we see nothing that seems off so we're going to look it at it from another perspective

we look at the response and see the error message:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

We could imagine that our valid user maybe has another error so we're going to use the grep match option:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

this will flag the response that contains this error message and so, will NOT flag the user that does not have this error:

and bingo:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

the **ao** user does not have our magic flag ->

we can see there is a dot missing:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we do the same thing and filter out the error message without the dot ->

<figure><img src="../../.gitbook/assets/image (5) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

and just like that, we solve this lab:

<figure><img src="../../.gitbook/assets/image (6) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

### Lab: Username enumeration via response timing

for this lab i first capture the req, launch intruder and sniper and add a known user in the word list:

<figure><img src="../../.gitbook/assets/image (7) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

And we get a timeout:

<figure><img src="../../.gitbook/assets/image (8) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

We later identify that the `X-Forwarded-For` header is supported, which allows you to spoof your IP address and bypass the IP-based brute-force protection.

{% embed url="https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For" %}

{% embed url="https://medium.com/r3d-buck3t/bypass-ip-restrictions-with-burp-suite-fb4c72ec8e9c" %}
