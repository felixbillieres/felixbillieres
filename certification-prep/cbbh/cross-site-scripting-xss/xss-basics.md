# 🧎‍♂️ XSS Basics

## Stored XSS

The most critical type of XSS vulnerability is `Stored XSS` or `Persistent XSS`. If our injected XSS payload gets stored in the back-end database and retrieved upon visiting the page, this means that our XSS attack is persistent and may affect any user that visits the page.

Let's imagine a blog or a todo list app where our input is pushed to the website ->

<figure><img src="../../../.gitbook/assets/image (1283).png" alt=""><figcaption></figcaption></figure>

We can test if vulnerable with this payload:

```html
<script>alert(window.origin)</script>
```

Another easy-to-spot payload is `<script>print()</script>` that will pop up the browser print dialog, which is unlikely to be blocked by any browsers or using `<plaintext>`, which will stop rendering the HTML code that comes after it and display it as plaintext.

_**To get the flag, use the same payload we used above, but change its JavaScript code to show the cookie instead of showing the url.**_

```
<script>alert(document.cookie)</script>
```

<figure><img src="../../../.gitbook/assets/image (1284).png" alt=""><figcaption></figcaption></figure>

## Reflected XSS

{% hint style="info" %}
There are two types of `Non-Persistent XSS` vulnerabilities: `Reflected XSS`, which gets processed by the back-end server, and `DOM-based XSS`, which is completely processed on the client-side and never reaches the back-end server.
{% endhint %}

`Reflected XSS` vulnerabilities occur when our input reaches the back-end server and gets returned to us without being filtered or sanitized. There are many cases in which our entire input might get returned to us, like error messages or confirmation messages.

In our input we just put 'test' to see what happens:

<figure><img src="../../../.gitbook/assets/image (1285).png" alt=""><figcaption></figcaption></figure>

We can see that `test` as part of the error message. If we test for XSS with the `<script>alert(window.origin)</script>` payload, we have the following:

<figure><img src="../../../.gitbook/assets/image (1286).png" alt=""><figcaption><p>It works</p></figcaption></figure>

If we visit the `Reflected` page again, the error message no longer appears, and our XSS payload is not executed, which means that this XSS vulnerability is indeed `Non-Persistent`.

To use this attack on a target we look at the network tab ->

<figure><img src="../../../.gitbook/assets/image (1287).png" alt=""><figcaption></figcaption></figure>

we can right-click on the `GET` request in the `Network` tab and select `Copy>Copy URL`. Once the victim visits this URL, the XSS payload would execute:

## DOM XSS
