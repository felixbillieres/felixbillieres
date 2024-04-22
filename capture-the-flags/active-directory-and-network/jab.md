# 🥊 Jab

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>





<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
nmap -sCV 10.129.69.162
Starting Nmap 7.93 ( https://nmap.org ) at 2024-04-03 19:14 BST
Nmap scan report for 10.129.69.162
Host is up (0.081s latency).
Not shown: 984 closed tcp ports (conn-refused)
PORT     STATE SERVICE             VERSION
53/tcp   open  domain              Simple DNS Plus
88/tcp   open  kerberos-sec        Microsoft Windows Kerberos (server time: 2024-04-03 18:14:36Z)
135/tcp  open  msrpc               Microsoft Windows RPC
139/tcp  open  netbios-ssn         Microsoft Windows netbios-ssn
389/tcp  open  ldap                Microsoft Windows Active Directory LDAP (Domain: jab.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-04-03T18:15:31+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=DC01.jab.htb
| Subject Alternative Name: othername:<unsupported>, DNS:DC01.jab.htb
| Not valid before: 2023-11-01T20:16:18
|_Not valid after:  2024-10-31T20:16:18
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http          Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap            Microsoft Windows Active Directory LDAP (Domain: jab.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.jab.htb
| Subject Alternative Name: othername:<unsupported>, DNS:DC01.jab.htb
| Not valid before: 2023-11-01T20:16:18
|_Not valid after:  2024-10-31T20:16:18
|_ssl-date: 2024-04-03T18:15:30+00:00; 0s from scanner time.
3268/tcp open  ldap                Microsoft Windows Active Directory LDAP (Domain: jab.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-04-03T18:15:31+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=DC01.jab.htb
| Subject Alternative Name: othername:<unsupported>, DNS:DC01.jab.htb
| Not valid before: 2023-11-01T20:16:18
|_Not valid after:  2024-10-31T20:16:18
3269/tcp open  ssl/ldap            Microsoft Windows Active Directory LDAP (Domain: jab.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.jab.htb
| Subject Alternative Name: othername:<unsupported>, DNS:DC01.jab.htb
| Not valid before: 2023-11-01T20:16:18
|_Not valid after:  2024-10-31T20:16:18
|_ssl-date: 2024-04-03T18:15:30+00:00; 0s from scanner time.
5222/tcp open  jabber              Ignite Realtime Openfire Jabber server 3.10.0 or later
| ssl-cert: Subject: commonName=dc01.jab.htb
| Subject Alternative Name: DNS:dc01.jab.htb, DNS:*.dc01.jab.htb
| Not valid before: 2023-10-26T22:00:12
|_Not valid after:  2028-10-24T22:00:12
|_ssl-date: TLS randomness does not represent time
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     features: 
|     capabilities: 
|     xmpp: 
|       version: 1.0
|     stream_id: 7ltzlqiziz
|     unknown: 
|     errors: 
|       invalid-namespace
|       (timeout)
|     auth_mechanisms: 
|_    compression_methods: 
5269/tcp open  xmpp                Wildfire XMPP Client
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     features: 
|     capabilities: 
|     xmpp: 
|     unknown: 
|     errors: 
|       (timeout)
|     auth_mechanisms: 
|_    compression_methods: 
7070/tcp open  realserver?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP: 
|     HTTP/1.1 400 Illegal character CNTL=0x0
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x0</pre>
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Wed, 03 Apr 2024 18:14:36 GMT
|     Last-Modified: Wed, 16 Feb 2022 15:55:02 GMT
|     Content-Type: text/html
|     Accept-Ranges: bytes
|     Content-Length: 223
|     <html>
|     <head><title>Openfire HTTP Binding Service</title></head>
|     <body><font face="Arial, Helvetica"><b>Openfire <a href="http://www.xmpp.org/extensions/xep-0124.html">HTTP Binding</a> Service</b></font></body>
|     </html>
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Wed, 03 Apr 2024 18:14:41 GMT
|     Allow: GET,HEAD,POST,OPTIONS
|   Help: 
|     HTTP/1.1 400 No URI
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 49
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: No URI</pre>
|   RPCCheck: 
|     HTTP/1.1 400 Illegal character OTEXT=0x80
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 71
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character OTEXT=0x80</pre>
|   RTSPRequest: 
|     HTTP/1.1 505 Unknown Version
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 58
|     Connection: close
|     <h1>Bad Message 505</h1><pre>reason: Unknown Version</pre>
|   SSLSessionReq: 
|     HTTP/1.1 400 Illegal character CNTL=0x16
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 70
|     Connection: close
|_    <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x16</pre>
7443/tcp open  ssl/oracleas-https?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP: 
|     HTTP/1.1 400 Illegal character CNTL=0x0
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x0</pre>
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Wed, 03 Apr 2024 18:14:42 GMT
|     Last-Modified: Wed, 16 Feb 2022 15:55:02 GMT
|     Content-Type: text/html
|     Accept-Ranges: bytes
|     Content-Length: 223
|     <html>
|     <head><title>Openfire HTTP Binding Service</title></head>
|     <body><font face="Arial, Helvetica"><b>Openfire <a href="http://www.xmpp.org/extensions/xep-0124.html">HTTP Binding</a> Service</b></font></body>
|     </html>
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Wed, 03 Apr 2024 18:14:48 GMT
|     Allow: GET,HEAD,POST,OPTIONS
|   Help: 
|     HTTP/1.1 400 No URI
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 49
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: No URI</pre>
|   RPCCheck: 
|     HTTP/1.1 400 Illegal character OTEXT=0x80
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 71
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character OTEXT=0x80</pre>
|   RTSPRequest: 
|     HTTP/1.1 505 Unknown Version
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 58
|     Connection: close
|     <h1>Bad Message 505</h1><pre>reason: Unknown Version</pre>
|   SSLSessionReq: 
|     HTTP/1.1 400 Illegal character CNTL=0x16
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 70
|     Connection: close
|_    <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x16</pre>
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc01.jab.htb
| Subject Alternative Name: DNS:dc01.jab.htb, DNS:*.dc01.jab.htb
| Not valid before: 2023-10-26T22:00:12
|_Not valid after:  2028-10-24T22:00:12
7777/tcp open  socks5              (No authentication; connection failed)
| socks-auth-info: 
|_  No authentication
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port7070-TCP:V=7.93%I=7%D=4/3%Time=660D9C8C%P=x86_64-pc-linux-gnu%r(Get
SF:Request,189,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Wed,\x2003\x20Apr\x2020
SF:24\x2018:14:36\x20GMT\r\nLast-Modified:\x20Wed,\x2016\x20Feb\x202022\x2
SF:015:55:02\x20GMT\r\nContent-Type:\x20text/html\r\nAccept-Ranges:\x20byt
SF:es\r\nContent-Length:\x20223\r\n\r\n<html>\n\x20\x20<head><title>Openfi
SF:re\x20HTTP\x20Binding\x20Service</title></head>\n\x20\x20<body><font\x2
SF:0face=\"Arial,\x20Helvetica\"><b>Openfire\x20<a\x20href=\"http://www\.x
SF:mpp\.org/extensions/xep-0124\.html\">HTTP\x20Binding</a>\x20Service</b>
SF:</font></body>\n</html>\n")%r(RTSPRequest,AD,"HTTP/1\.1\x20505\x20Unkno
SF:wn\x20Version\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nConte
SF:nt-Length:\x2058\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x205
SF:05</h1><pre>reason:\x20Unknown\x20Version</pre>")%r(HTTPOptions,56,"HTT
SF:P/1\.1\x20200\x20OK\r\nDate:\x20Wed,\x2003\x20Apr\x202024\x2018:14:41\x
SF:20GMT\r\nAllow:\x20GET,HEAD,POST,OPTIONS\r\n\r\n")%r(RPCCheck,C7,"HTTP/
SF:1\.1\x20400\x20Illegal\x20character\x20OTEXT=0x80\r\nContent-Type:\x20t
SF:ext/html;charset=iso-8859-1\r\nContent-Length:\x2071\r\nConnection:\x20
SF:close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20c
SF:haracter\x20OTEXT=0x80</pre>")%r(DNSVersionBindReqTCP,C3,"HTTP/1\.1\x20
SF:400\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;c
SF:harset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\
SF:r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x
SF:20CNTL=0x0</pre>")%r(DNSStatusRequestTCP,C3,"HTTP/1\.1\x20400\x20Illega
SF:l\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;charset=iso-88
SF:59-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x2
SF:0Message\x20400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x0</p
SF:re>")%r(Help,9B,"HTTP/1\.1\x20400\x20No\x20URI\r\nContent-Type:\x20text
SF:/html;charset=iso-8859-1\r\nContent-Length:\x2049\r\nConnection:\x20clo
SF:se\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20No\x20URI</pre>
SF:")%r(SSLSessionReq,C5,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL
SF:=0x16\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Lengt
SF:h:\x2070\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><
SF:pre>reason:\x20Illegal\x20character\x20CNTL=0x16</pre>");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port7443-TCP:V=7.93%T=SSL%I=7%D=4/3%Time=660D9C92%P=x86_64-pc-linux-gnu
SF:%r(GetRequest,189,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Wed,\x2003\x20Apr
SF:\x202024\x2018:14:42\x20GMT\r\nLast-Modified:\x20Wed,\x2016\x20Feb\x202
SF:022\x2015:55:02\x20GMT\r\nContent-Type:\x20text/html\r\nAccept-Ranges:\
SF:x20bytes\r\nContent-Length:\x20223\r\n\r\n<html>\n\x20\x20<head><title>
SF:Openfire\x20HTTP\x20Binding\x20Service</title></head>\n\x20\x20<body><f
SF:ont\x20face=\"Arial,\x20Helvetica\"><b>Openfire\x20<a\x20href=\"http://
SF:www\.xmpp\.org/extensions/xep-0124\.html\">HTTP\x20Binding</a>\x20Servi
SF:ce</b></font></body>\n</html>\n")%r(HTTPOptions,56,"HTTP/1\.1\x20200\x2
SF:0OK\r\nDate:\x20Wed,\x2003\x20Apr\x202024\x2018:14:48\x20GMT\r\nAllow:\
SF:x20GET,HEAD,POST,OPTIONS\r\n\r\n")%r(RTSPRequest,AD,"HTTP/1\.1\x20505\x
SF:20Unknown\x20Version\r\nContent-Type:\x20text/html;charset=iso-8859-1\r
SF:\nContent-Length:\x2058\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Messa
SF:ge\x20505</h1><pre>reason:\x20Unknown\x20Version</pre>")%r(RPCCheck,C7,
SF:"HTTP/1\.1\x20400\x20Illegal\x20character\x20OTEXT=0x80\r\nContent-Type
SF::\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2071\r\nConnectio
SF:n:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illega
SF:l\x20character\x20OTEXT=0x80</pre>")%r(DNSVersionBindReqTCP,C3,"HTTP/1\
SF:.1\x20400\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/
SF:html;charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20clos
SF:e\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20chara
SF:cter\x20CNTL=0x0</pre>")%r(DNSStatusRequestTCP,C3,"HTTP/1\.1\x20400\x20
SF:Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;charset=
SF:iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>
SF:Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=
SF:0x0</pre>")%r(Help,9B,"HTTP/1\.1\x20400\x20No\x20URI\r\nContent-Type:\x
SF:20text/html;charset=iso-8859-1\r\nContent-Length:\x2049\r\nConnection:\
SF:x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20No\x20URI
SF:</pre>")%r(SSLSessionReq,C5,"HTTP/1\.1\x20400\x20Illegal\x20character\x
SF:20CNTL=0x16\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nContent
SF:-Length:\x2070\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400
SF:</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x16</pre>");
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-04-03T18:15:23
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.56 seconds
```

We see port 88 open, we can fire up pre-auth:

{% content-ref url="../../active-directory/attacking-vectors/initial-attack-vectors/preauth-user-enumeration.md" %}
[preauth-user-enumeration.md](../../active-directory/attacking-vectors/initial-attack-vectors/preauth-user-enumeration.md)
{% endcontent-ref %}

```
kerbrute userenum --dc dc01.jab.htb -d jab.htb /usr/share/SecLists/Usernames/xato-net-10-million-usernames.txt
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

seeing the number of users, we will probably do a password spraying

We see a xmpp server running, while looking at how we interract with it on internet i encounter the nmap documentation:

{% embed url="https://nmap.org/nsedoc/scripts/xmpp-brute.html" %}

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

it did not work
