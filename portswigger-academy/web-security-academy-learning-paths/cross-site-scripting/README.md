# 🎌 Cross-site scripting

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

There are three main types of XSS attacks. These are:

* [Reflected XSS](https://portswigger.net/web-security/cross-site-scripting#reflected-cross-site-scripting), where the malicious script comes from the current HTTP request.
* [Stored XSS](https://portswigger.net/web-security/cross-site-scripting#stored-cross-site-scripting), where the malicious script comes from the website's database.
* [DOM-based XSS](https://portswigger.net/web-security/cross-site-scripting#dom-based-cross-site-scripting), where the vulnerability exists in client-side code rather than server-side code.

**Reflected:**

It arises when an application receives data in an HTTP request and includes that data within the immediate response in an unsafe way.

ex:

```
https://insecure-website.com/status?message=All+is+well.
<p>Status: All is well.</p>
```

We could then construct the following:

```
https://insecure-website.com/status?message=<script>/*+Bad+stuff+here...+*/</script>
<p>Status: <script>/* Bad stuff here... */</script></p>
```

Cheat sheet:

[Cross-site scripting cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

**Stored:**

It's when an application receives data from an untrusted source and includes that data within its later HTTP responses in an unsafe way.

Let's imagine a message board input for the users:

```
<p>Hello, this is my message!</p>
```

We could possibly create a payload:

```
<p><script>/* Bad stuff here... */</script></p>
```

**DOM-based:**

It's when an application contains some client-side JavaScript that processes data from an untrusted source in an unsafe way, usually by writing the data back to the DOM.

If the application uses some JavaScript to read the value from an input field and write that value to an element within the HTML like this:

```
var search = document.getElementById('search').value;
var results = document.getElementById('results');
results.innerHTML = 'You searched for: ' + search;
```

As an attacker, we could easily a value that executes our malicious script:

```
You searched for: <img src=1 onerror='/* Bad stuff here... */'>
```
