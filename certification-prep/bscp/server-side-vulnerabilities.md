# 🈯 Server-side vulnerabilities

Path traversal is also known as directory traversal. These vulnerabilities enable an attacker to read arbitrary files on the server that is running an application.

A vulnerable application would look something like this:

```
<img src="/loadImage?filename=218.png">    
```

and we'd use `../../`  to read content we are not meant to read:

```
https://insecure-website.com/loadImage?filename=../../../etc/passwd
```

### Lab: File path traversal, simple case

I go on a product page:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

i see where the image is fetched from:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I capture a request when loading the image and modify the path to go and fetch another file:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and I am able to see the passwd file:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Access Control

Access control is the application of constraints on who or what is authorized to perform actions or access resources.

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

if a non-administrative user can gain access to an admin page where they can delete user accounts, then this is vertical privilege escalation.

### Lab: Unprotected admin functionality

looking at the website:

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

after enumeration, or a gobuster we find a /robots.txt:

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we then go on the admin panel and are able to delete users:

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If the application uses "security by obscurity", or more complex URL's it might not be discoverable by  gobuster or directory bruteforce like:

```
https://insecure-website.com/administrator-panel-yb556
```

However, the application might still leak the URL to users. The URL might be disclosed in JavaScript that constructs the user interface based on the user's role:

```
<script>
	var isAdmin = false;
	if (isAdmin) {
		...
		var adminPanelTag = document.createElement('a');
		adminPanelTag.setAttribute('https://insecure-website.com/administrator-panel-yb556');
		adminPanelTag.innerText = 'Admin panel';
		...
	}
</script>
```

### Lab: Unprotected admin functionality with unpredictable URL

We get a new website, i go and check the source code immediately:

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and bingo:

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Parameter-based access control methods

Some applications determine the user's access rights or role at login, and then store this information in a user-controllable location. This could be:

* A hidden field.
* A cookie.
* A preset query string parameter.

```
https://insecure-website.com/login/home.jsp?admin=true
https://insecure-website.com/login/home.jsp?role=1
```

a user can modify the value and access functionality they're not authorized to, such as administrative functions.

### Lab: User role controlled by request parameter

i connect with the given credentials wiener:peter and go to the given admin panel `/admin`

we see he cookies:

<figure><img src="../../.gitbook/assets/image (802).png" alt=""><figcaption></figcaption></figure>

We modify the value to true and refresh the page:

<figure><img src="../../.gitbook/assets/image (803).png" alt=""><figcaption></figcaption></figure>

bingo

#### Horizontal privilege escalation

Horizontal privilege escalation occurs if a user is able to gain access to resources belonging to another user

```
https://insecure-website.com/myaccount?id=123
```

we could change the admin id to 124 and maybe access to another users panel

### Lab: User ID controlled by request parameter, with unpredictable user IDs

<figure><img src="../../.gitbook/assets/image (804).png" alt=""><figcaption></figcaption></figure>

we connect:

<figure><img src="../../.gitbook/assets/image (805).png" alt=""><figcaption></figcaption></figure>

while looking around we find an article written by carlos:

<figure><img src="../../.gitbook/assets/image (822).png" alt=""><figcaption></figcaption></figure>

when clicking on the name, we see the url change:

<figure><img src="../../.gitbook/assets/image (823).png" alt=""><figcaption><p>2c732f45-bd3b-4ae8-9fe3-5f6bd75219c2</p></figcaption></figure>

and on our space we can see that the page is based on our ID:

<figure><img src="../../.gitbook/assets/image (824).png" alt=""><figcaption></figcaption></figure>

so what if we change the values ->

<figure><img src="../../.gitbook/assets/image (825).png" alt=""><figcaption></figcaption></figure>

we get the value of carlos and are able to solve the lab

### Lab: User ID controlled by request parameter with password disclosure

after connecting with the given creds, we immediatly see :thumbsup:

<figure><img src="../../.gitbook/assets/image (826).png" alt=""><figcaption></figcaption></figure>

the "id=wiener" seems to be the way to go, let's modify it->

<figure><img src="../../.gitbook/assets/image (827).png" alt=""><figcaption></figcaption></figure>

we are able to see to the admin page and extract the password if we look at the source code ->

we are then able to connect to the admin account and delete user carlos:

<figure><img src="../../.gitbook/assets/image (828).png" alt=""><figcaption></figcaption></figure>

#### Authentication vulnerabilities

Authentication vulnerabilities can allow attackers to gain access to sensitive data and functionality.

<figure><img src="../../.gitbook/assets/image (829).png" alt=""><figcaption></figcaption></figure>

### Lab: Username enumeration via different responses

we got 2 lists, usernames and passwords, the best way would be a cluster bomb attack but without burp Pro it would be very long

We stick to sniper attack by only selecting usernames and throw in the attack to see if we can identify a valid user with response code or length ->

all of the status codes are 200 but one of the Length is not the same:

<figure><img src="../../.gitbook/assets/image (830).png" alt=""><figcaption></figcaption></figure>

let's try with users ads

<figure><img src="../../.gitbook/assets/image (831).png" alt=""><figcaption></figcaption></figure>

we put the password as the target of this new sniper attack and launch the attack with the wordlist ->

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we just have to wait and pick up a response code or length and lab is solved

### Lab: 2FA simple bypass

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ok so when i connect to my account, i am able to see the verification code and login

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>my-account?id=wiener</p></figcaption></figure>

i then try to connect with carlos account cred:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

the catch here is even if we connected and are requested a verif code, we are in fact already logged in and if I were to replace the URL with the expected URL after inputting the 2FA code, i'll have access to everything:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Server-side request forgery

It's a web security vulnerability that allows an attacker to cause the server-side application to make requests to an unintended location.

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Lab: Basic SSRF against the local server

We do not have any creds, while looking on the website we see this odd button:

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we also see we can't access admin panel except for a few conditions:

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we capture it:

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

when we capture the request:

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We modify the api request to do a loopback:

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and in the source code we see the url we need to access to delete user carlos:

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we then request this URL via the API and solve the lab:

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Lab: Basic SSRF against another back-end system

We start by capturing the request:

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

according to the exercise i have to find an admin panel on a network so i start by making a request to /admin:8080 but put the network in a sniper attack position:

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we finally find the url that gives us a status code 200:

<figure><img src="../../.gitbook/assets/image (17) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and when we request the admin control panel we get a 200:

<figure><img src="../../.gitbook/assets/image (18) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

in the source code we see the delete carlos request:

<figure><img src="../../.gitbook/assets/image (19) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we just request it through the stockAPI and lab solved :thumbsup:

#### File upload vulnerabilities

File upload vulnerabilities are when a web server allows users to upload files to its filesystem without sufficiently validating things like their name, type, contents, or size.

### Lab: Remote code execution via web shell upload

we start by uploading an image to the server:

<figure><img src="../../.gitbook/assets/image (20) (1) (1).png" alt=""><figcaption></figcaption></figure>

We see this ouput in the burp request:

<figure><img src="../../.gitbook/assets/image (21) (1).png" alt=""><figcaption></figcaption></figure>

We have to modify a few things to read content:

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

### Lab: Web shell upload via Content-Type restriction bypass

we start by uploading an image FP.png and putting our malicious php code in it ->

and everything works:

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

#### OS command injection

OS command injection is also known as shell injection. It allows an attacker to execute operating system (OS) commands on the server that is running an application, and typically fully compromise the application and its data. Often, an attacker can leverage an OS command injection vulnerability to compromise other parts of the hosting infrastructure, and exploit trust relationships to pivot the attack to other systems within the organization.

### Lab: OS command injection, simple case

We start by going on a product and checking the stock and sending it to burp:

sending it to repeater and pipe whoami the storeID parameter:

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

and find the user just like that

#### SQL injection

SQL injection (SQLi) is a web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database. This can allow an attacker to view data that they are not normally able to retrieve. This might include data that belongs to other users, or any other data that the application can access. In many cases, an attacker can modify or delete this data, causing persistent changes to the application's content or behavior.

In some situations, an attacker can escalate a SQL injection attack to compromise the underlying server or other back-end infrastructure. It can also enable them to perform denial-of-service attacks.

### Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

We simply input 'OR 1=1-- - in the category section:

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

### Lab: SQL injection vulnerability allowing login bypass

This one was straightforward

just had to input in the username field: wiener'0R 1=1-- -

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>
