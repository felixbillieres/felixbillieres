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
