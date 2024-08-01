# 📥 Databases

Web applications utilize back end [databases](https://en.wikipedia.org/wiki/Database) to store various content and information related to the web application. This can be core web application assets like images and files, web application content like posts and updates, or user data like usernames and passwords.

Here are some different types of databases:

### Relational (SQL)

[Relational](https://en.wikipedia.org/wiki/Relational\_database) (SQL) databases store their data in tables, rows, and columns. Each table can have unique keys, which can link tables together and create relationships between tables.

<figure><img src="../../../../.gitbook/assets/image (6) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

We can link the `id` from the `users` table to the `user_id` in the `posts` table to easily retrieve the user details for each post, without having to store all user details with each post.

Here is a list if common relational databases:

<table><thead><tr><th width="141">Type</th><th>Description</th></tr></thead><tbody><tr><td><a href="https://en.wikipedia.org/wiki/MySQL">MySQL</a></td><td>The most commonly used database around the internet. It is an open-source database and can be used completely free of charge</td></tr><tr><td><a href="https://en.wikipedia.org/wiki/Microsoft_SQL_Server">MSSQL</a></td><td>Microsoft's implementation of a relational database. Widely used with Windows Servers and IIS web servers</td></tr><tr><td><a href="https://en.wikipedia.org/wiki/Oracle_Database">Oracle</a></td><td>A very reliable database for big businesses, and is frequently updated with innovative database solutions to make it faster and more reliable. It can be costly, even for big businesses</td></tr><tr><td><a href="https://en.wikipedia.org/wiki/PostgreSQL">PostgreSQL</a></td><td>Another free and open-source relational database. It is designed to be easily extensible, enabling adding advanced new features without needing a major change to the initial database design</td></tr></tbody></table>

### Non-relational (NoSQL)

A [non-relational database](https://en.wikipedia.org/wiki/NoSQL) does not use tables, rows, columns, primary keys, relationships, or schemas. Instead, a `NoSQL` database stores data using various storage models, depending on the type of data stored.

There are 4 common storage models for `NoSQL` databases:

* Key-Value
* Document-Based
* Wide-Column
* Graph

Each of the above models has a different way of storing data. For example, the `Key-Value` model usually stores data in `JSON` or `XML`, and has a key for each pair, storing all of its data as its value:

<figure><img src="../../../../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here are some common NoSql databses:

<table><thead><tr><th width="214">Type</th><th>Description</th></tr></thead><tbody><tr><td><a href="https://en.wikipedia.org/wiki/MongoDB">MongoDB</a></td><td>The most common <code>NoSQL</code> database. It is free and open-source, uses the <code>Document-Based</code> model, and stores data in <code>JSON</code> objects</td></tr><tr><td><a href="https://en.wikipedia.org/wiki/Elasticsearch">ElasticSearch</a></td><td>Another free and open-source <code>NoSQL</code> database. It is optimized for storing and analyzing huge datasets. As its name suggests, searching for data within this database is very fast and efficient</td></tr><tr><td><a href="https://en.wikipedia.org/wiki/Apache_Cassandra">Apache Cassandra</a></td><td>Also free and open-source. It is very scalable and is optimized for gracefully handling faulty values</td></tr></tbody></table>

Practical use of a PHP web app with a MySQL DB:

once `MySQL` is up and running, we can connect to the database server with:

```php
$conn = new mysqli("localhost", "user", "pass");
```

Then, we can create a new database with:

```php
$sql = "CREATE DATABASE database1";
$conn->query($sql)
```

After that, we can connect to our new database, and start using the `MySQL` database through `MySQL` syntax, right within `PHP`, as follows:

```php
$conn = new mysqli("localhost", "user", "pass", "database1");
$query = "select * from table_1";
$result = $conn->query($query);
```
