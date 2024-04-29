# Authentication vulnerabilities

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

There are 3 main type of authentication:

* Something you **know**, such as a password or the answer to a security question. These are sometimes called "knowledge factors".
* Something you **have**, This is a physical object such as a mobile phone or security token. These are sometimes called "possession factors".
* Something you **are** or do. For example, your biometrics or patterns of behavior. These are sometimes called "inherence factors".

It's crucial to know the difference between **authentication and authorization**

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

### Lab: Username enumeration via different responses

We first start by taking the given list and triggering all the usernames so we can bypass the cluster bomb that will take very long without the pro version:&#x20;

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

We get user ads with a response way different from the rest, let's go on that road ->

We then trigger the list of password with username ads and see that the good password gives us a 302 response →

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
