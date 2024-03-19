# 🤠 Intro to Dante

<figure><img src="../../../../.gitbook/assets/image (463).png" alt=""><figcaption><p>all the boxes in this intro</p></figcaption></figure>

### Emdee five for life

<figure><img src="../../../../.gitbook/assets/image (464).png" alt=""><figcaption><p>I suppose I need to auto submit it </p></figcaption></figure>

### Heist

{% embed url="https://app.hackthebox.com/machines/201" %}

<figure><img src="../../../../.gitbook/assets/image (465).png" alt=""><figcaption></figcaption></figure>

First we're redirected to this page:

<figure><img src="../../../../.gitbook/assets/image (466).png" alt=""><figcaption></figcaption></figure>

after a gobuster we find a **issues.php**

<figure><img src="../../../../.gitbook/assets/image (467).png" alt=""><figcaption></figcaption></figure>

it's a login page but there is login as guest button that leads us to this page. Let's click on attachments&#x20;

<figure><img src="../../../../.gitbook/assets/image (468).png" alt=""><figcaption></figcaption></figure>

thanks to hash type, i see the pattern "$1$" and see that its probably:

<figure><img src="../../../../.gitbook/assets/image (470).png" alt=""><figcaption></figcaption></figure>

So i can try to use hash cat with mode = 500 or john this way ->

<figure><img src="../../../../.gitbook/assets/image (469).png" alt=""><figcaption></figcaption></figure>

we find this "stealthagent" result

and with online decoder we found t
