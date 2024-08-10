# 🦩 Login Brute Forcing

Here is a small list of files that can contain hashed passwords:

| **`Windows`** | **`Linux`** |
| ------------- | ----------- |
| unattend.xml  | shadow      |
| sysprep.inf   | shadow.bak  |
| SAM           | password    |

There are many tools and methods to utilize for login brute-forcing, like:

* `Ncrack`
* `wfuzz`
* `medusa`
* `patator`
* `hydra`
* and others.

Many web servers or individual contents on the web servers are still often used with the [Basic HTTP AUTH](https://tools.ietf.org/html/rfc7617) scheme.

The HTTP specification provides two parallel authentication mechanisms:

1. `Basic HTTP AUTH` is used to authenticate the user to the HTTP server.
2. `Proxy Server Authentication` is used to authenticate the user to an intermediate proxy server.

The Basic HTTP Authentication scheme uses user ID and password for authentication. The client sends a request without authentication information with its first request. The server's response contains the `WWW-Authenticate` header field, which requests the client to provide the credentials.

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

Here are the different password attacks:

| **Password Attack Type** |
| ------------------------ |
| `Dictionary attack`      |
| `Brute force`            |
| `Traffic interception`   |
| `Man In the Middle`      |
| `Key Logging`            |
| `Social engineering`     |
