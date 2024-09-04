# 🔗 HTTP Verb Tampering

Suppose both the web application and the back-end web server are configured only to accept `GET` and `POST` requests. In that case, sending a different request will cause a web server error page to be displayed, which is not a severe vulnerability in itself. if the web server configurations are not restricted to only accept the HTTP methods required by the web server (e.g. `GET`/`POST`), and the web application is not developed to handle other types of HTTP requests (e.g. `HEAD`, `PUT`), then we may be able to exploit this insecure configuration to gain access to functionalities we do not have access to, or even bypass certain security controls.

HTTP has [9 different verbs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) that can be accepted as HTTP methods by web servers.

Here are the most known:

| Verb      | Description                                                                                         |
| --------- | --------------------------------------------------------------------------------------------------- |
| `HEAD`    | Identical to a GET request, but its response only contains the `headers`, without the response body |
| `PUT`     | Writes the request payload to the specified location                                                |
| `DELETE`  | Deletes the resource at the specified location                                                      |
| `OPTIONS` | Shows different options accepted by a web server, like accepted HTTP verbs                          |
| `PATCH`   | Apply partial modifications to the resource at the specified location                               |

## Bypassing Basic Authentication

### Identify

Let's say we got the following web application:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and when we want to reset everythong we are prompted with a password.

Since we don't have any credentials, we will get a `401 Unauthorized` page.

First thing to do is identify which pages are restricted, we can see that reset button navigates to `/admin/reset.php`. So, either the `/admin` directory is restricted to authenticated users only, or only the `/admin/reset.php` page is. We can confirm this by visiting the `/admin` directory, and we do indeed get prompted to log in again. This means that the full `/admin` directory is restricted.

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

We can see that it's a GET method, we can try changing it to POST but we still get prompted with credentials. We could try using the `HEAD` method, which is identical to a `GET` request but does not return the body in the HTTP response. If this is successful, we may not receive any output, but the `reset` function should still get executed

```shell-session
ElFelixi0@htb[/htb]$ curl -i -X OPTIONS http://SERVER_IP:PORT/

HTTP/1.1 200 OK
```

So this means the web server indeed accepts `HEAD` requests, which is the default configuration for many web servers.

Once we change `POST` to `HEAD` and forward the request, we will see that we no longer get a login prompt or a `401 Unauthorized` page and get an empty output instead

<figure><img src="../../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

```
Try to use what you learned in this section to access the 'reset.php' page and delete all files. Once all files are deleted, you should get the flag. 
```

<figure><img src="../../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

And once we forward ->

<figure><img src="../../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

## Bypassing Security Filters

### Identify

if we try to create a new file name with special characters in its name (e.g. `test;`), we get the following message ->

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This message shows that the web application uses certain filters on the back-end to identify injection attempts and then blocks any malicious requests.

We can try to intercept the request in Burp Suite (Burp) and then use `Change Request Method` to change it to another method

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we can try if we can inject a command that creates two files and then check whether both files were created. To do so, we will use the following file name in our attack (`file1; touch file2;`)

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**To get the flag, try to bypass the command injection filter through HTTP Verb Tampering, while using the following filename: file; cp /flag.txt ./**_

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
