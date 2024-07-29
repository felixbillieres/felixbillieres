# 🧜 SQL Injection Fundamentals

A SQL injection occurs when a malicious user attempts to pass input that changes the final SQL query sent by the web application to the database, enabling the user to perform other unintended SQL queries directly against the database.

## **MySQL Syntax**

### Relational & Non-relational Databases

#### Relational Databases:

1. **Structure**:
   * Use a structured schema with tables, rows, and columns.
   * Each table has a predefined structure, and relationships between tables are defined using foreign keys.
2. **Storage**:
   * Store data in a tabular format.
   * SQL (Structured Query Language) is used for querying and managing the data.
3. **Use Cases**:
   * Suitable for applications with complex queries and transactions, such as financial systems, enterprise resource planning (ERP) systems, and customer relationship management (CRM) systems.
   * Ideal for scenarios requiring ACID (Atomicity, Consistency, Isolation, Durability) properties.

#### Non-relational Databases:

1. **Structure**:
   * Use a flexible schema that can store various data formats, such as documents, key-value pairs, graphs, or wide-columns.
   * Do not require a predefined structure, allowing for more flexibility and scalability.
2. **Storage**:
   * Store data in formats like JSON, BSON, XML, or other non-tabular formats.
   * Different querying languages depending on the database type (e.g., MongoDB uses a query language similar to JSON).
3. **Use Cases**:
   * Suitable for applications with large volumes of unstructured or semi-structured data, such as social media platforms, big data applications, and real-time analytics.
   * Ideal for scenarios where horizontal scalability and fast read/write operations are crucial.

### MySQL

To connect to mysql/mariaDB database, here is the command ->

```
mysql -u root -p
```

When we do not specify a host, it will default to the `localhost` server. We can specify a remote host and port using the `-h` and `-P` flags.

```
mysql -u root -h docker.hackthebox.eu -P 3306 -p 
```

If we wanted to create a database:

```sql
mysql> CREATE DATABASE users;
#And to show the DBs ->
mysql> SHOW DATABASES;
```

To create a table:

```sql
CREATE TABLE logins (
    id INT,
    #If We want to add one automatically: id INT NOT NULL AUTO_INCREMENT,
    username VARCHAR(100),
    #If we want no username used twice: username VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(100),
    date_of_joining DATETIME
    #takes automatically the time atm: date_of_joining DATETIME DEFAULT NOW(),
    );
    
#And to show the table:
mysql> SHOW TABLES;

#If we wanted to get more infos about the table:
mysql> DESCRIBE logins;
```

If we want to retrieve data, we can use SELECT statement:

```sql
SELECT * FROM table_name;
```

We can also select specific columns from a table:

```sql
SELECT column1, column2 FROM table_name;
```

If we wanted to remove the tabke and databases from the server we could use the DROP command:

```sql
mysql> DROP TABLE logins;
```

we can [LIMIT](https://dev.mysql.com/doc/refman/8.0/en/limit-optimization.html) the results to what we want only, using `LIMIT` and the number of records we want

```sql
mysql> SELECT * FROM logins LIMIT 2;
```

To filter or search for specific data, we can use conditions with the `SELECT` statement using the [WHERE](https://dev.mysql.com/doc/refman/8.0/en/where-optimization.html) clause

```sql
SELECT * FROM logins WHERE id > 1;
SELECT * FROM logins where username = 'admin';
```

Another useful SQL clause is [LIKE](https://dev.mysql.com/doc/refman/8.0/en/pattern-matching.html), enabling selecting records by matching a certain pattern.

```sql
SELECT * FROM logins WHERE username LIKE 'admin%';
# or all usernames with exactly three characters in them
SELECT * FROM logins WHERE username like '___';
```

If we want to print out conditions to be more specific:

```sql
SELECT * FROM logins WHERE username != 'john' AND id > 1;
SELECT * FROM logins WHERE username != 'tom' AND id > 3 - 2;
SELECT * FROM logins WHERE username != 'tom' AND id > 1;

```

_**In the 'titles' table, what is the number of records WHERE the employee number is greater than 10000 OR their title does NOT contain 'engineer'?**_

```sql
select * from titles where title NOT LIKE '%engineer%' or emp_no > 10000;
```
