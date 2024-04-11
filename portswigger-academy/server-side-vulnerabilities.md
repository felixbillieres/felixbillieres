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

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

i see where the image is fetched from:

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

I capture a request when loading the image and modify the path to go and fetch another file:

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

and I am able to see the passwd file:

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

#### Access Control

Access control is the application of constraints on who or what is authorized to perform actions or access resources.

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

if a non-administrative user can gain access to an admin page where they can delete user accounts, then this is vertical privilege escalation.

### Lab: Unprotected admin functionality

looking at the website:

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

after enumeration, or a gobuster we find a /robots.txt:

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

we then go on the admin panel and are able to delete users:

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

and bingo:

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (802).png" alt=""><figcaption></figcaption></figure>

We modify the value to true and refresh the page:

<figure><img src="../.gitbook/assets/image (803).png" alt=""><figcaption></figcaption></figure>

bingo

#### Horizontal privilege escalation

Horizontal privilege escalation occurs if a user is able to gain access to resources belonging to another user

```
https://insecure-website.com/myaccount?id=123
```

we could change the admin id to 124 and maybe access to another users panel

### Lab: User ID controlled by request parameter, with unpredictable user IDs

<figure><img src="../.gitbook/assets/image (804).png" alt=""><figcaption></figcaption></figure>

we connect:

<figure><img src="../.gitbook/assets/image (805).png" alt=""><figcaption></figcaption></figure>

we find the csrf token value of our user:

<figure><img src="../.gitbook/assets/image (806).png" alt=""><figcaption></figcaption></figure>

