---
description: https://app.hackthebox.com/machines/143
---

# 🎣 Reel

<figure><img src="../../.gitbook/assets/image (1035).png" alt=""><figcaption></figcaption></figure>

Here is the nmap output of the scan:

```
PORT      STATE SERVICE      VERSION
21/tcp    open  ftp          Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_05-29-18  12:19AM       <DIR>          documents
22/tcp    open  ssh          OpenSSH 7.6 (protocol 2.0)
| ssh-hostkey: 
|   2048 8220c3bd16cba29c88871d6c1559eded (RSA)
|   256 232bb80a8c1cf44d8d7e5e6458803345 (ECDSA)
|_  256 ac8bde251db7d838389b9c16bff63fed (ED25519)
25/tcp    open  smtp?
| smtp-commands: REEL, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, X11Probe: 
|     220 Mail Service ready
|   FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, RTSPRequest: 
|     220 Mail Service ready
|     sequence of commands
|     sequence of commands
|   Hello: 
|     220 Mail Service ready
|     EHLO Invalid domain address.
|   Help: 
|     220 Mail Service ready
|     DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
|   SIPOptions: 
|     220 Mail Service ready
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|   TerminalServerCookie: 
|     220 Mail Service ready
|_    sequence of commands
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2012 R2 Standard 9600 microsoft-ds (workgroup: HTB)
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49159/tcp open  msrpc        Microsoft Windows RPC
```

The odd bit is that the ftp port indicates it's a windows box and the ssh port would let us think it's a linux box so maybe there is a VM techno behind all that

We can assume it's a windows box after a quick ping since default TTL on windows is 127 compared to 64 for linux

<figure><img src="../../.gitbook/assets/image (1036).png" alt=""><figcaption></figcaption></figure>

We connect through anonymous login on the ftp server and go get all the documents:

<figure><img src="../../.gitbook/assets/image (1037).png" alt=""><figcaption></figcaption></figure>

We go through the documents and see interesting stuff letting us think there's maybe a bot reviewing emails for a potential phishing chal, and via exiftool we find a potential username:

<figure><img src="../../.gitbook/assets/image (1038).png" alt=""><figcaption><p>nico@megabank.com</p></figcaption></figure>

And just under we can see the software used is:

```
Application                     : Microsoft Office Word
```

Now we are going to go through port 25 which is the smtp server and communicate with the mail service ->

`HELO felix.com`: The `HELO` command is used to introduce the client to the server. Here, `felix.com` is the domain name of the client.

`MAIL FROM: <felix@felix.com>`: This command specifies the email address of the sender.

<figure><img src="../../.gitbook/assets/image (1039).png" alt=""><figcaption></figcaption></figure>

Thanks to this little experiment, we can confirm that user nico exists because SMTP validates users coming from outside the server but does not validate when we try with a user that "should" be in the server database

* `RCPT TO: <nico@megabank.com>`: The `RCPT TO` command specifies the recipient’s email address. The server responds with `250 OK`, indicating that the recipient is valid.
* This process is repeated for multiple recipients. The server responds with `250 OK` for `nico@megabank.com`, `iWantMyOSCP@please.com`, and `OSCP2024@megabank.local`, indicating these recipients are valid.
* `RCPT TO: <helloworld@megabank.com>`: When specifying this recipient, the server responds with `550 Unknown user`, indicating that the email address `helloworld@megabank.com` does not exist on the server.
