# 🫁 Server-side request forgery (SSRF)

Server-side request forgery is a web security vulnerability that allows an attacker to cause the server-side application to make requests to an unintended location.

<figure><img src="../../.gitbook/assets/image (17) (1) (1).png" alt=""><figcaption></figcaption></figure>

Let's say we have a _check stock_ option on the website that makes this request:

```
stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```

If we change it to this:

```
stockApi=http://localhost/admin
```

It would fetch the content of admin and return it to the user if misconfigured because the application may trust a request that comes from a local machine because of misconfigured trust relationships

### Lab: Basic SSRF against the local server

There is a check stock on all the different products, if we capture a request we get this:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We change the request to :

```
http://localhost/admin
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>don't forget to URL encode it</p></figcaption></figure>

and in the response we have access to the admin panel of the application and the request needed to delete a user:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

To achieve this challenge, the final request will be:

```
stockApi=http%3a//localhost/admin/delete?username=carlos
```

#### SSRF attacks against other back-end systems

If the IP address is a non-routable private IP address, we could think that, since the backend server thinks he's protected by the network topology, the security is weaker and lets anyone able to interact with it admin privileges

### Lab: Basic SSRF against another back-end system

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we fetch the request and send it to intruder:

I'll cheat a bit and say we probably have the same request in the first exercise for this exercise:

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We launch the attack and wait:

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<pre><code><strong>stockApi=http%3a//192.168.0.49%3a8080/admin/delete%3fusername%3dcarlos
</strong></code></pre>

we don't forget to URL encode it and then forward it, and bingo the lab is solved

**SSRF with blacklisted input filters:**

To bypass the fact that some applications block `127.0.0.1` and `localhost`, or sensitive URLs like `/admin` here are 2 ways that i think are very nice to bypass it:

* Use an alternative IP representation of `127.0.0.1`, such as `2130706433`, `017700000001`, or `127.1`.\*

1. **Decimal Representation (2130706433)**:
   * IP addresses are typically represented in a dotted decimal format, where each octet is represented by a decimal number ranging from 0 to 255. However, internally, IP addresses are stored as binary numbers.
   * The decimal representation of an IP address can be converted to its binary form. In the case of 127.0.0.1, its binary representation is 01111111000000000000000000000001.
   * Using the decimal representation (2130706433) of the loopback address (127.0.0.1) might bypass filters that specifically look for the dotted decimal format.
2. **Octal Representation (017700000001)**:
   * In some programming languages like C and Python, you can represent numbers in octal notation by prefixing them with '0'.
   * The octal representation of 127.0.0.1 is 017700000001, where each octet is represented in octal form.
   * This representation could be used to evade filters that only check for the decimal or dotted decimal format of IP addresses.
3. **Shortened Representation (127.1)**:
   * This representation simply omits the trailing zeroes in the dotted decimal format. It's still valid and points to the same IP address (127.0.0.1).
     * Filters or security mechanisms that only look for specific patterns, such as the exact match of "127.0.0.1", might overlook variations like "127.1", allowing attackers to bypass them.

* Register your own domain name that resolves to `127.0.0.1`. You can use `spoofed.burpcollaborator.net` for this purpose.

### Lab: SSRF with blacklist-based input filter

So we start by capturing the request of the check stock button and try to input a localhost bypass but fail:

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>note: URL encoded</p></figcaption></figure>

Ok so we are able to bypass the input restriction with the shortened version of 127.0.0.1:

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

but /admin did not work, so i tried encoding a few times and it worked:

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

1. **Encoded URL**: `http%3a//127.1/%2561dmin`
   * `%3a` represents the colon character `:` in URL encoding.
   * `%25` represents the percent character `%` itself. So, `%2561` actually decodes to `%61`, which represents the lowercase letter `a` in ASCII.
2. **Decoded URL**: `http://127.1/%61dmin`
   * The URL now reads `http://127.1/admin`.

So now i simply look at the admin page and look for the Carlos delete user button and send the request to solve the lab:

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**Open redirection vulnerability**

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Open redirection vulnerability occurs when a web application redirects users to a URL specified by user-supplied input, without proper validation. Attackers can exploit this vulnerability to redirect users to malicious websites, phishing pages, or other harmful destinations.

Let's say we have an application that has this vuln with the following URL:

```
/product/nextProduct?currentProductId=6&path=http://evil-user.net
```

that obviously returns to `http://evil-user.net`

We can leverage this to bypass the URL filter and exploit SSRF:

```
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin
```

{% hint style="info" %}
This SSRF exploit works because the application first validates that the supplied `stockAPI` URL is on an allowed domain, which it is. The application then requests the supplied URL, which triggers the open redirection.
{% endhint %}

### Lab: SSRF with filter bypass via open redirection vulnerability

I got a bit stuck on the initial foothold of the machine, then I saw this odd "next item" button that could mean a redirection to another page:

<figure><img src="../../.gitbook/assets/image (14) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Okay, so I found this redirection error:

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

It seems to redirect just fine so we go back on the check stock API and throw in the inital payload to access the admin page:

<figure><img src="../../.gitbook/assets/image (16) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And we're able to access the page to delete user:

<figure><img src="../../.gitbook/assets/image (17) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**How to find blind SSRF?**

The most reliable way to detect blind SSRF vulnerabilities is using out-of-band (OAST) techniques

<figure><img src="../../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

### Lab: Blind SSRF with out-of-band detection

I don't have burp pro unfortunately, so I can't do it

{% embed url="https://portswigger.net/web-security/learning-paths/ssrf-attacks/ssrf-attacks-blind-ssrf-vulnerabilities/ssrf/blind/lab-out-of-band-detection" %}
