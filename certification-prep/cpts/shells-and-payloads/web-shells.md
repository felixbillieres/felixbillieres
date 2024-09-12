# ✨ Web Shells

A `web shell` is a browser-based shell session we can use to interact with the underlying operating system of a web server. Our goal will be to obtain an RCE or file disclosure

## Laudanum

Laudanum is a repository of ready-made files that can be used to inject onto a victim and receive back access via a reverse shell

The repo includes injectable files for many different web application languages to include `asp, aspx, jsp, php,` and more.

We first need to copy a shell we intend to use in our current directory ->

```shell-session
cp /usr/share/laudanum/aspx/shell.aspx /home/tester/demo.aspx
```

We then need to change te IP to our localhost&#x20;

<figure><img src="../../../.gitbook/assets/image (1450).png" alt=""><figcaption></figcaption></figure>

We then find a vulnerable app with an upload function ->

<figure><img src="../../../.gitbook/assets/image (1451).png" alt=""><figcaption></figcaption></figure>

We can see where the file was uploaded so we can just navigate to this repo ->

<figure><img src="../../../.gitbook/assets/image (1452).png" alt=""><figcaption></figcaption></figure>

And we will be able to enumerate the web app in our browser

## Antak Webshell

`Active Server Page Extended` (`ASPX`) is a file type/extension written for [Microsoft's ASP.NET Framework](https://docs.microsoft.com/en-us/aspnet/overview). On a web server running the ASP.NET framework, web form pages can be generated for users to input data. On the server side, the information will be converted into HTML. We can take advantage of this by using an ASPX-based web shell to control the underlying Windows operating system

Antak is a web shell built-in ASP.Net included within the [Nishang project](https://github.com/samratashok/nishang). Antak utilizes PowerShell to interact with the host, making it great for acquiring a web shell on a Windows server.

The Antak files can be found in the `/usr/share/nishang/Antak-WebShell` directory.

To use it, we first need to copy it to our current directory

```shell-session
cp /usr/share/nishang/Antak-WebShell/antak.aspx /home/administrator/Upload.aspx
```

We then modify the shell:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

When we navigate to the page for use. It will give you a user and password prompt.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And then once we enter our credentials we should see a powershell theme webshell

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## PHP Web Shells

"PHP is used by `78.6%` of all websites whose server-side programming language we know

Let's say we have the follwoing page:\


<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Since PHP processes code & commands on the server-side, we can use pre-written payloads to gain a shell through the browser or initiate a reverse shell session with our attack box.

We come accross this upload form:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We are going to use [WhiteWinterWolf's PHP Web Shell](https://github.com/WhiteWinterWolf/wwwolf-php-webshell)

But we quickly see that we can't upload this file because of our extension so we hop on burp -<

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We will change Content-type from `application/x-php` to `image/gif`. And Forward it and our file upload was successful. Now we can use the browser, navigate to this directory on the rConfig server:

`/images/vendor/connect.php`

This executes the payload and provides us with a non-interactive shell session entirely in the browser

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
