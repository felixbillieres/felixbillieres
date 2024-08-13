# 🐻 HTTP Fundamentals

## HyperText Transfer Protocol (HTTP)

HTTP communication consists of a client and a server, where the client requests the server for a resource. The server processes the requests and returns the requested resource.

HTTP's port is port 80 but it can change depending on the web server configuration.

The process to fetch an http ressource is the following: we enter a `Fully Qualified Domain Name` (`FQDN`) as a `Uniform Resource Locator` (`URL)`

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The main mandatory fields are the scheme and the host, without which the request would have no resource to request.

Here is the anatomy of a HTTP request:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### cURL

cURL does not render the HTML/JavaScript/CSS code, unlike a web browser, but prints it in its raw format.

```
ElFelixio@htb[/htb]$ curl inlanefreight.com

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
...SNIP...
```

If we ever wanted to download a page or a file and output the content into a file using the `-O` flag. If we want to specify the output file name, we can use the `-o` flag and specify the name.

### **Questions**

To get the flag, start the above exercise, then use cURL to download the file returned by '/download.php' in the server shown above.

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Hypertext Transfer Protocol Secure (HTTPS)

one of the significant drawbacks of HTTP is that all data is transferred in clear-text so it's very vulnerable to MITM attacks like we can see in the follwing:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and here is a pcap file of a HTTPS login:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here is the flow of how HTTPS operates:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### cURL

If the website we're curling does not have a valid SSL certificate, the curl command would not work, to bypass this check we can use the -k flag

```
ElFelixio@htb[/htb]$ curl https://inlanefreight.com

curl: (60) SSL certificate problem: Invalid certificate chain
More details here: https://curl.haxx.se/docs/sslcerts.html
...SNIP...

#and here we bypass the check:

ElFelixio@htb[/htb]$ curl -k https://inlanefreight.com

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
...SNIP...
```

## HTTP Requests and Responses

### HTTP Request

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The image above shows an HTTP GET request to the URL:

* `http://inlanefreight.com/users/login.html`

The next set of lines contain HTTP header value pairs, like `Host`, `User-Agent`, `Cookie`, and many other possible headers. These headers are used to specify various attributes of a request.

### HTTP Response

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Response codes are used to determine the request's status

It's good to know that cURL also allows us to preview the full HTTP request and the full HTTP response if we add `-v` flag  which can become very handy when performing web penetration tests or writing exploits.

```
ElFelixio@htb[/htb]$ curl inlanefreight.com -v

*   Trying SERVER_IP:80...
* TCP_NODELAY set
* Connected to inlanefreight.com (SERVER_IP) port 80 (#0)
> GET / HTTP/1.1
> Host: inlanefreight.com
> User-Agent: curl/7.65.3
> Accept: */*
> Connection: close
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 401 Unauthorized
< Date: Tue, 21 Jul 2020 05:20:15 GMT
< Server: Apache/X.Y.ZZ (Ubuntu)
< WWW-Authenticate: Basic realm="Restricted Content"
< Content-Length: 464
< Content-Type: text/html; charset=iso-8859-1
< 
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>

...SNIP...
```

## HTTP Headers

HTTP headers pass information between the client and the server. Some headers are only used with either requests or responses, while some other general headers are common to both.

Here are the 5 types of headers:

1. `General Headers`
2. `Entity Headers`
3. `Request Headers`
4. `Response Headers`
5. `Security Headers`

### General Headers

[General headers](https://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html) are used in both HTTP requests and responses, it's used to describe the message rather than its contents

### Entity Headers

[Entity Headers](https://www.w3.org/Protocols/rfc2616/rfc2616-sec7.html) can be `common to both the request and response`. These headers are used to `describe the content` (entity) transferred by a message. They are usually found in responses and POST or PUT requests.

### Request Headers

The client sends [Request Headers](https://tools.ietf.org/html/rfc2616) in an HTTP transaction. These headers are `used in an HTTP request and do not relate to the content` of the message.

### Response Headers

[Response Headers](https://tools.ietf.org/html/rfc7231#section-6) can be `used in an HTTP response and do not relate to the content`. Certain response headers such as `Age`, `Location`, and `Server` are used to provide more context about the response.

### Security Headers

HTTP Security headers are `a class of response headers used to specify certain rules and policies` to be followed by the browser while accessing the website.

***

If we wanted to curl only the header of the request, we would use the -I flag to send a HEAD reuest and display the response header and the -i flag to display both the header and the body of the req
