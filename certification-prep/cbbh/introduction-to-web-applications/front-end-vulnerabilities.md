# Front End Vulnerabilities

## Sensitive Data Exposure

[Sensitive Data Exposure](https://owasp.org/www-project-top-ten/2017/A3\_2017-Sensitive\_Data\_Exposure) refers to the availability of sensitive data in clear-text to the end-user.

It can be utilized to gain access to sensitive functionality (i.e., an admin panel), which may lead to compromising the entire server.

#### Example:

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

If there is no sanitizing and we input this malicious code:

```html
<style> body { background-image: url('https://academy.hackthebox.com/images/logo.svg'); } </style>
```

The website will look like this:

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

## Cross-Site Scripting (XSS)

