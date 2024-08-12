# ☑️ Session Security

A user session can be defined as a sequence of requests originating from the same client and the associated responses during a specific time period.

A unique **session identifier** (Session ID) or token is the basis upon which user sessions are generated and distinguished so if an attacker obtains a session identifier, this can result in session hijacking.

session identifier's security level depends on `Validity Scope, Randomness, Validity Time`

It also depends on where it is stored `(URL, HTML, sessionStorage, localStorage)`

Here is the commands used to connect to the vhosts:

```shell-session
ElFelixi0@htb[/htb]$ IP=ENTER SPAWNED TARGET IP HERE
ElFelixi0@htb[/htb]$ printf "%s\t%s\n\n" "$IP" "xss.htb.net csrf.htb.net oredirect.htb.net minilab.htb.net" | sudo tee -a /etc/hosts
```
