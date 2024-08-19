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

While `reflected XSS` sends the input data to the back-end server through HTTP requests, DOM XSS is completely processed on the client-side through JavaScript. DOM XSS occurs when JavaScript is used to change the page source through the `Document Object Model (DOM)`.

Let's imagine we send our input via the todo list app and open up network tab ->

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

this is a client-side parameter that is completely processed on the browser.

we must understand the concept of the `Source` and `Sink` of the object displayed on the page. The `Source` is the JavaScript object that takes the user input, and it can be any input parameter like a URL parameter or an input field, as we saw above.

On the other hand, the `Sink` is the function that writes the user input to a DOM Object on the page. If the `Sink` function does not properly sanitize the user input, it would be vulnerable to an XSS attack.

Some of the commonly used JavaScript functions to write to DOM objects are:

* `document.write()`
* `DOM.innerHTML`
* `DOM.outerHTML`

Furthermore, some of the `jQuery` library functions that write to DOM objects are:

* `add()`
* `after()`
* `append()`

If we check `script.js` and see  the page uses functions like the `innerHTML` function to write the `task` variable in the `todo` DOM

```javascript
document.getElementById("todo").innerHTML = "<b>Next Task:</b> " + decodeURIComponent(task);
```

we can see that we can control the input, and the output is not being sanitized, so this page should be vulnerable to DOM XSS.

Our previous payload does not work because the `innerHTML` function does not allow the use of the `<script>`

We can start and look at other payloads such as&#x20;

```html
<img src="" onerror=alert(window.origin)>
```

The above line creates a new HTML image object, which has a `onerror` attribute that can execute JavaScript code when the image is not found. So, as we provided an empty image link (`""`), our code should always get executed without having to use `<script>` tags

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

## XSS Discovery

Some of the common open-source tools that can assist us in XSS discovery are [XSS Strike](https://github.com/s0md3v/XSStrike), [Brute XSS](https://github.com/rajeshmajumdar/BruteXSS), and [XSSer](https://github.com/epsylon/xsser)

```shell-session
ElFelixi0@htb[/htb]$ git clone https://github.com/s0md3v/XSStrike.git
ElFelixi0@htb[/htb]$ cd XSStrike
ElFelixi0@htb[/htb]$ pip install -r requirements.txt
ElFelixi0@htb[/htb]$ python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test" 
```

e can find huge lists of XSS payloads online, like the one on [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md) or the one in [PayloadBox](https://github.com/payloadbox/xss-payload-list).

_**Utilize some of the techniques mentioned in this section to identify the vulnerable input parameter found in the above server. What is the name of the vulnerable parameter?**_

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>
