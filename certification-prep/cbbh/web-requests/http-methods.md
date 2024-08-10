# 🐰 HTTP Methods

## Basic knowledge

Here are the different methods for accessing & interracting with  a ressource

|            |                                                                                                                                                                                                                                                                                                                                      |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Method** | **Description**                                                                                                                                                                                                                                                                                                                      |
| `GET`      | Requests a specific resource. Additional data can be passed to the server via query strings in the URL (e.g. `?param=value`).                                                                                                                                                                                                        |
| `POST`     | Sends data to the server. It can handle multiple types of input, such as text, PDFs, and other forms of binary data. This data is appended in the request body present after the headers. The POST method is commonly used when sending information (e.g. forms/logins) or uploading data to a website, such as images or documents. |
| `HEAD`     | Requests the headers that would be returned if a GET request was made to the server. It doesn't return the request body and is usually made to check the response length before downloading resources.                                                                                                                               |
| `PUT`      | Creates new resources on the server. Allowing this method without proper controls can lead to uploading malicious resources.                                                                                                                                                                                                         |
| `DELETE`   | Deletes an existing resource on the webserver. If not properly secured, can lead to Denial of Service (DoS) by deleting critical files on the web server.                                                                                                                                                                            |
| `OPTIONS`  | Returns information about the server, such as the methods accepted by it.                                                                                                                                                                                                                                                            |
| `PATCH`    | Applies partial modifications to the resource at the specified location.                                                                                                                                                                                                                                                             |

Normal web apps rely on GET and POST but it's good to know that web application that utilizes REST APIs also rely on `PUT` and `DELETE`, which are used to update and delete data on the API endpoint

When going for a ressource on a webserver HTTP status codes are used to tell the client the status of their request. Here are the fives categories:

| **Type** | **Description**                                                                                                                  |
| -------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `1xx`    | Provides information and does not affect the processing of the request.                                                          |
| `2xx`    | Returned when a request succeeds.                                                                                                |
| `3xx`    | Returned when the server redirects the client.                                                                                   |
| `4xx`    | Signifies improper requests `from the client`. For example, requesting a resource that doesn't exist or requesting a bad format. |
| `5xx`    | Returned when there is some problem `with the HTTP server` itself.                                                               |

## GET

Let's say we have a login page where we had to login:

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

When we log in we have access to the following ip:

```
http://<SERVER_IP>:<PORT>/
```

But obviously if we try to curl this ip without logging in we'll have access denied ->

```
ElFelixio@htb[/htb]$ curl -i http://<SERVER_IP>:<PORT>/
HTTP/1.1 401 Authorization Required
Date: Mon, 21 Feb 2022 13:11:46 GMT
Server: Apache/2.4.41 (Ubuntu)
Cache-Control: no-cache, must-revalidate, max-age=0
WWW-Authenticate: Basic realm="Access denied"
Content-Length: 13
Content-Type: text/html; charset=UTF-8

Access denied
```

To bypass this problem we can use the -u flag to specify credentials in our command:

```
ElFelixio@htb[/htb]$ curl -u admin:admin http://<SERVER_IP>:<PORT>/

<!DOCTYPE html>
<html lang="en">

<head>
...SNIP...
```

Another way of doing this is to specify the credentials in the url:

```
ElFelixio@htb[/htb]$ curl http://admin:admin@<SERVER_IP>:<PORT>/

<!DOCTYPE html>
<html lang="en">

<head>
```

If we try and use the -v for more informations about our request, we can see some base64:

```
ElFelixio@htb[/htb]$ curl -v http://admin:admin@<SERVER_IP>:<PORT>/

*   Trying <SERVER_IP>:<PORT>...
* Connected to <SERVER_IP> (<SERVER_IP>) port PORT (#0)
* Server auth using Basic with user 'admin'
> GET / HTTP/1.1
> Host: <SERVER_IP>
> Authorization: Basic YWRtaW46YWRtaW4=
> User-Agent: curl/7.77.0
> Accept: */*

...
```

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### POST

So for this exercise we are going to look at what requests are made when we login with admin:admin ->

<figure><img src="../../../.gitbook/assets/image (15) (2).png" alt=""><figcaption></figcaption></figure>

And when we login we can see some odd stuff:

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

If we go in the Request tab and look at the raw data we can see our credentials:

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

If we wanted to look at it from our CLI we could just right click and copy as cURL:

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

But it's good to know how to craft this on our own:

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

We will use the `-X POST` flag to send a `POST` request. Then, to add our POST data, we can use the `-d` flag

```bash
curl http://94.237.53.113:34314/ -X -d admin:admin
```

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption><p>and it works</p></figcaption></figure>

or this one:

```bash
curl -X POST -d 'username=admin&password=admin' http://94.237.53.113:34314/
```

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Many login forms would redirect us to a different page once authenticated (e.g. /dashboard.php). If we want to follow the redirection with cURL, we can use the `-L` flag.
{% endhint %}

{% hint style="info" %}
If we were successfully authenticated, we should have received a cookie so our browsers can persist our authentication, and we don't need to login every time we visit the page. We can use the `-v` or `-i` flags to view the response, which should contain the `Set-Cookie` header with our authenticated cookie:
{% endhint %}

```bash
curl -X POST -d 'username=admin&password=admin' http://94.237.53.113:34314/ -i
```

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

With our session cookie we can interract with the web app without any credentials with the -b flag:

```bash
curl -b 'PHPSESSID=1sc2lgt4jbnd1ii0skpq9u5dob' http://94.237.53.113:34314/
```

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

```
#This would also work as is specify's the cookie as a header:
curl -H 'Cookie: PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' http://94.237.53.113:34314/
```

we could also show this as we manually log out and replace the new cookie with our old authenticated cookie and refresh the page to connect to the search panel:

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Now let's look at the JSON data being sent when we interract with the search button:

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

Raw data looks like: `{"search":"Tokyo"}*`

The POST data appear to be in JSON format, so our request must have specified the `Content-Type` header to be `application/json`. We can confirm this by right-clicking on the request, and selecting `Copy>Copy Request Headers`:

we could curl this information with this command:

```shell-session
curl -X POST -d '{"search":"london"}' -b 'PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' -H 'Content-Type: application/json' http://<SERVER_IP>:<PORT>/search.php
```

Let's try what we just did with `Fetch`. We can right-click on the request and select `Copy>Copy as Fetch`, and then go to the `Console` tab and execute the code:

```
await fetch("http://94.237.53.113:34314/search.php", {
    "credentials": "include",
    "headers": {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; rv:109.0) Gecko/20100101 Firefox/115.0",
        "Accept": "*/*",
        "Accept-Language": "en-US,en;q=0.5",
        "Content-Type": "application/json",
        "Sec-GPC": "1"
    },
    "referrer": "http://94.237.53.113:34314/",
    "body": "{\"search\":\"Tokyo\"}",
    "method": "POST",
    "mode": "cors"
});
```

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

Obtain a session cookie through a valid login, and then use the cookie with cURL to search for the flag through a JSON POST request to '/search.php'

```
curl -X POST -d '{"search":"flag"}' -b 'PHPSESSID=1sc2lgt4jbnd1ii0skpq9u5dob' -H 'Content-Type: application/json' http://94.237.53.113:34314/search.php
```

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

## CRUD API

Many APIs are used to interact with a database, such that we would be able to specify the requested table and the requested row within our API query, and then use an HTTP method to perform the operation needed.

A cURL request could look something like:

```
curl -X PUT http://<SERVER_IP>:<PORT>/api.php/city/london ...SNIP...
```

In general, APIs perform 4 main operations on the requested database entity:

| Operation | HTTP Method | Description                                        |
| --------- | ----------- | -------------------------------------------------- |
| `Create`  | `POST`      | Adds the specified data to the database table      |
| `Read`    | `GET`       | Reads the specified entity from the database table |
| `Update`  | `PUT`       | Updates the data of the specified database table   |
| `Delete`  | `DELETE`    | Removes the specified row from the database table  |

### Read

```
curl -s http://83.136.254.58:46803/api.php/city/london
```

<figure><img src="../../../.gitbook/assets/image (9) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

For a better output, we can pipe jq at the end of our command:

<figure><img src="../../../.gitbook/assets/image (10) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

We quickly realise that we can see all of the cities that are in the api when we don't specify any city and pipe jq:

<figure><img src="../../../.gitbook/assets/image (11) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

### Create

```
curl -X POST http://83.136.254.58:46803/api.php/city/ -d '{"city_name":"ElFelixioLand", "country_name":"HTB"}' -H 'Content-Type: application/json' | jq
```

We can craft our own city and post it to the api and check if it was succesfull:

<figure><img src="../../../.gitbook/assets/image (12) (1) (2).png" alt=""><figcaption></figcaption></figure>

### Update

We can alter the api's content and update it with the following commands:

```
curl -X PUT http://83.136.254.58:46803/api.php/city/london -d '{"city_name":"ElLondonito", "country_name":"ParisIsBetter"}' -H 'Content-Type: application/json'
```

And check if it was really updated:

<figure><img src="../../../.gitbook/assets/image (13) (2).png" alt=""><figcaption></figcaption></figure>

### DELETE

This one is very simple and is just like a READ command:

```
curl -X DELETE http://83.136.254.58:46803/api.php/city/ElFelixioLand
```

<figure><img src="../../../.gitbook/assets/image (14) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1272).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1273).png" alt=""><figcaption></figcaption></figure>

```
curl http://94.237.59.63:34518/search.php?search=flag -u admin:admin
```

## POST

whenever web applications need to transfer files or move the user parameters from the URL, they utilize `POST` requests

`POST` places user parameters within the HTTP Request body.

sO we get this login page:

<figure><img src="../../../.gitbook/assets/image (1274).png" alt=""><figcaption></figcaption></figure>
