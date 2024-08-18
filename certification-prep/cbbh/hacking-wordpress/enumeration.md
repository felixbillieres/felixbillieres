# 🍍 Enumeration

## Version & Themes/Plugins Enumeration

To start our enumeration, we can go and look at the source code for the  `meta generator` tag using the shortcut `[CTRL + F]` or grep the output with curl ->

```html
<SNIP>
<meta name="generator" content="WordPress 5.3.3" />
<SNIP>
```

```shell-session
curl -s -X GET http://blog.inlanefreight.com | grep '<meta name="generator"'
```

Enumeration for plugins installed ->

```shell-session
curl -s -X GET http://blog.inlanefreight.com | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'wp-content/plugins/*' | cut -d"'" -f2
```

Enumeration for themes ->

```shell-session
curl -s -X GET http://blog.inlanefreight.com | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'themes' | cut -d"'" -f2
```

not all installed plugins and themes can be discovered passively. We can send a GET request that points to a directory or file that may exist on the server.

```shell-session
curl -I -X GET http://blog.inlanefreight.com/wp-content/plugins/mail-masta
```

If the content does not exist, we will receive a `404 Not Found error`.

## Directory Indexing

Even if a plugin is deactivated, it may still be accessible, and therefore we can gain access to its associated scripts and functions ->

<figure><img src="../../../.gitbook/assets/image (1406).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1407).png" alt=""><figcaption><p>we can see that we still have access to the <code>Mail Masta</code> plugin</p></figcaption></figure>

Or with curl ->

```shell-session
curl -s -X GET http://blog.inlanefreight.com/wp-content/plugins/mail-masta/ | html2text
```

## Enumeration

The first method is reviewing posts to uncover the ID assigned to the user and their corresponding username. If we mouse over the post author link titled "by admin,"

<figure><img src="../../../.gitbook/assets/image (1408).png" alt=""><figcaption></figcaption></figure>

To check if it's really id=1, we can curl the response or go straight in the URL ->

```
curl -s -I -X GET http://blog.inlanefreight.com/?author=1
```

And also check for non existing users ->

```shell-session
curl -s -I -X GET http://blog.inlanefreight.com/?author=100
```

Before version 4.7.1 we could get the list of the users by interracting with the JSON ->&#x20;

```shell-session
ElFelixi0@htb[/htb]$ curl http://blog.inlanefreight.com/wp-json/wp/v2/users | jq

[
  {
    "id": 1,
    "name": "admin",
    "url": "",
    "description": "",
    "link": "http://blog.inlanefreight.com/index.php/author/admin/",
    <SNIP>
  },
  {
    "id": 2,
    "name": "ch4p",
    "url": "",
    "description": "",
    "link": "http://blog.inlanefreight.com/index.php/author/ch4p/",
    <SNIP>
  },
  <SNIP>
```

Now it shows whether a user is configured or not.

## Login

Once we got our users, we can mount a password brute-forcing attack to attempt to gain access to the WordPress backend. This attack can be performed via the login page or the `xmlrpc.php` page.

If our POST request against `xmlrpc.php` contains valid credentials, we will receive the following output:

```shell-session
ElFelixi0@htb[/htb]$ curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>CORRECT-PASSWORD</value></param></params></methodCall>" http://blog.inlanefreight.com/xmlrpc.php

<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
  <params>
    <param>
      <value>
      <array><data>
  <value><struct>
  <member><name>isAdmin</name><value><boolean>1</boolean></value></member>
  <member><name>url</name><value><string>http://blog.inlanefreight.com/</string></value></member>
  <member><name>blogid</name><value><string>1</string></value></member>
  <member><name>blogName</name><value><string>Inlanefreight</string></value></member>
  <member><name>xmlrpc</name><value><string>http://blog.inlanefreight.com/xmlrpc.php</string></value></member>
</struct></value>
</data></array>
      </value>
    </param>
  </params>
</methodResponse>
```

If not we receive `403`

_**Search for "WordPress xmlrpc attacks" and find out how to use it to execute all method calls. Enter the number of possible method calls of your target as the answer.**_

First we capture the request of the xmlrpc.php page ->

<figure><img src="../../../.gitbook/assets/image (1409).png" alt=""><figcaption></figcaption></figure>

Now we send to repeater, change to POST and add the following in the body to list all methods allowded ->

```
<methodCall>
<methodName>system.listMethods</methodName>
<params></params>
</methodCall>
```

<figure><img src="../../../.gitbook/assets/image (1410).png" alt=""><figcaption></figcaption></figure>

Then to count how many methods are in this, we devide by 2 the total string count:

<figure><img src="../../../.gitbook/assets/image (1411).png" alt=""><figcaption></figcaption></figure>

## WPScan

install:

```shell-session
gem install wpscan
```

normal enumeration scan:

```shell-session
wpscan --url http://blog.inlanefreight.com --enumerate --api-token Kffr4fdJzy9qVcTk<SNIP>
```

_**Enumerate the provided WordPress instance for all installed plugins. Perform a scan with WPScan against the target and submit the version of the vulnerable plugin named “photo-gallery”.**_

```
wpscan --url http://94.237.59.63:47915 --enumerate p
```

<figure><img src="../../../.gitbook/assets/image (1412).png" alt=""><figcaption></figcaption></figure>
