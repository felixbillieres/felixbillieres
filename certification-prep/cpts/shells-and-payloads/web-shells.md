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
