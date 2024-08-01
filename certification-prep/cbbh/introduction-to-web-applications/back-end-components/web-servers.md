# 🎅 Web Servers

A [web server](https://en.wikipedia.org/wiki/Web\_server) is an application that runs on the back end server, which handles all of the HTTP traffic from the client-side browser, routes it to the requested pages, and finally responds to the client-side browser.

Here is the workflow of a web server:

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

A typical web server accepts HTTP requests from the client-side, and responds with different HTTP responses and codes, like a code `200 OK` response for a successful request, a code `404 NOT FOUND` when requesting pages that do not exist, code `403 FORBIDDEN` for requesting access to restricted pages, and so on.

You can find the most common [HTTP response codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) on the link

Here are some of the most used web servers in the world:

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

[Apache](https://www.apache.org/) 'or `httpd`' is the most common web server on the internet, hosting more than `40%` of all internet websites. `Apache` usually comes pre-installed in most `Linux` distributions and can also be installed on Windows and macOS servers.

`Apache` is usually used with `PHP` for web application development, but it also supports other languages like `.Net`, `Python`, `Perl`, and even OS languages like `Bash` through `CGI`.

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

[NGINX](https://www.nginx.com/) is the second most common web server on the internet, hosting roughly `30%` of all internet websites. `NGINX` focuses on serving many concurrent web requests with relatively low memory and CPU load by utilizing an async architecture to do so.

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

[IIS (Internet Information Services)](https://en.wikipedia.org/wiki/Internet\_Information\_Services) is the third most common web server on the internet, hosting around `15%` of all internet web sites. `IIS` is developed and maintained by Microsoft and mainly runs on Microsoft Windows Servers.

`IIS` is very well optimized for Active Directory integration and includes features like `Windows Auth` for authenticating users using Active Directory, allowing them to automatically sign in to web applications.
