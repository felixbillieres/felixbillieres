# 🍏 Web Proxy

I will not go on how to set up foxy proxy and capture a request, i'll hop right in the exploitation and testing part:

Let's say we have this web app:

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We capture the request after sending inputting an IP:

```html
POST /ping HTTP/1.1
Host: 46.101.23.188:30820
Content-Length: 4
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://46.101.23.188:30820
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://46.101.23.188:30820/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

ip=1
```

now our goal is to "break" the application ("breaking" the request/response flow by manipulating the target parameter, not damaging the target web application). If the web application does not verify and validate the HTTP requests on the back-end, we may be able to manipulate it and exploit it.

What will happen if we change the requets from ip=1 to `;ls;`

```
...
Accept-Language: en-US,en;q=0.9
Connection: close

ip=;ls;
```

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**Try intercepting the ping request on the server shown above, and change the post data similarly to what we did in this section. Change the command to read 'flag.txt'**_

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Intercepting Responses

We may want to intercept the HTTP responses from the server before they reach the browser.

In our previous exercise, the `IP` field only allowed us to input numeric values. If we intercept the response before it reaches our browser, we can edit it to accept any value, which would enable us to input the payload we used last time directly.

In Burp, we can enable response interception by going to (`Proxy>Options`) and enabling `Intercept Response` under `Intercept Server Responses`:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here is the normal HTML of the web app we were exploiting:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

Let's try changing the `type="number"` on line 27 to `type="text"` and change the input length to be able to type bigger commands:

```
<input type="text" id="ip" name="ip" min="1" max="255" maxlength="100"
    oninput="javascript: if (this.value.length > this.maxLength) this.value = this.value.slice(0, this.maxLength);"
    required>
```

After forwarding the request, we can now input some text and commands:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

## Automatic Modification

We can choose to match any text within our requests, either in the request header or request body, and then replace them with different text. We can go to (`Proxy>Options>Match and Replace`) and click on `Add` in Burp.

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

After clicking ok, this will start automatically replacing the `User-Agent` header in our requests with our new User-Agent. We can verify that by visiting any website using the pre-configured Burp browser and reviewing the intercepted request.

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

## Repeating Requests

To go faster with exploitation and not being in the need to recapture a request every time, we can utilize request repeating to make this process significantly easier.&#x20;

To start, we can view the HTTP requests history in `Burp` at (`Proxy>HTTP History`):

If we click on a previous request such as:

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

We can now send it to the repeater (`CTRL+R),` edit our request and hit the send button

<figure><img src="../../../.gitbook/assets/image (9) (1) (2).png" alt=""><figcaption></figcaption></figure>

_**Try using request repeating to be able to quickly test commands. With that, try looking for the other flag.**_

<figure><img src="../../../.gitbook/assets/image (10) (1) (2).png" alt=""><figcaption></figcaption></figure>

## Encoding/Decoding

Here are some common errors that can occure if we do not encode our data:

* `Spaces`: May indicate the end of request data if not encoded
* `&`: Otherwise interpreted as a parameter delimiter
* `#`: Otherwise interpreted as a fragment identifier

To encode any data in burp you can select (`Convert Selection>URL>URL encode key characters`), or by selecting the text and clicking \[`CTRL+U`]

In recent versions of Burp, we can also use the `Burp Inspector` tool to perform encoding and decoding (among other things), which can be found in various places like `Burp Proxy` or `Burp Repeater`:

In recent versions of Burp, we can also use the `Burp Inspector` tool to perform encoding and decoding (among other things), which can be found in various places like `Burp Proxy` or `Burp Repeater`:

## Proxying Tools

### Proxychains

proxychains routes all traffic coming from any command-line tool to any proxy we specify. `Proxychains` adds a proxy to any command-line tool and is hence the simplest and easiest method to route web traffic of command-line tools through our web proxies.

To use `proxychains`, we first have to edit `/etc/proxychains.conf`, comment out the final line and add the following line at the end of it:

```wasm
#socks4         127.0.0.1 9050
http 127.0.0.1 8080
```

We can now try to use cURL on the previous web app:

```shell-session
ElFelixio@htb[/htb]$ proxychains curl http://SERVER_IP:PORT

ProxyChains-3.1 (http://proxychains.sf.net)
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Ping IP</title>
    <link rel="stylesheet" href="./style.css">
```

### Nmap

With nmap we can use the `--proxies` flag. We should also add the `-Pn` flag to skip host discovery (as recommended on the man page). Finally, we'll also use the `-sC` flag to examine what an nmap script scan does

```shell-session
ElFelixio@htb[/htb]$ nmap --proxies http://127.0.0.1:8080 SERVER_IP -pPORT -Pn -sC

Starting Nmap 7.91 ( https://nmap.org )
Nmap scan report for SERVER_IP
Host is up (0.11s latency).

PORT      STATE SERVICE
PORT/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 0.49 seconds
```

### Metasploit

We can start Metasploit with `msfconsole`. Then, to set a proxy for any exploit within Metasploit, we can use the `set PROXIES` flag

```sh
ElFelixio@htb[/htb]$ msfconsole

msf6 > use auxiliary/scanner/http/robots_txt
msf6 auxiliary(scanner/http/robots_txt) > set PROXIES HTTP:127.0.0.1:8080

PROXIES => HTTP:127.0.0.1:8080


msf6 auxiliary(scanner/http/robots_txt) > set RHOST SERVER_IP

RHOST => SERVER_IP


msf6 auxiliary(scanner/http/robots_txt) > set RPORT PORT

RPORT => PORT


msf6 auxiliary(scanner/http/robots_txt) > run

[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

and then we can go back to our web proxy tool of choice and examine the proxy history to view all sent requests:

<figure><img src="../../../.gitbook/assets/image (11) (1) (2).png" alt=""><figcaption></figcaption></figure>

_**Try running 'auxiliary/scanner/http/http\_put' in Metasploit on any website, while routing the traffic through Burp. Once you view the requests sent, what is the last line in the request?**_

