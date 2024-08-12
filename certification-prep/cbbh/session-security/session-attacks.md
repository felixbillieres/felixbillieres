# Session Attacks

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
