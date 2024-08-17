# 🦘 Bypassing Filters

Many web applications only rely on front-end JavaScript code to validate the selected file format before it is uploaded

we can easily bypass it by directly interacting with the server, skipping the front-end validations altogether. We may also modify the front-end code through our browser's dev tools to disable any validation in place.

Let's say we have an upload image form that blocks our .php file:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Back-end Request Modification

We start by sending the request to burp:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The important thing to look at is `filename="HTB.png"` and the file content at the end of the request. If we modify the `filename` to `shell.php` and modify the content to the web shell we used in the previous section; we would be uploading a `PHP` web shell instead of an image.

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p><code>File successfully uploaded</code></p></figcaption></figure>

### Disabling Front-end Validation

Let's start by \[`CTRL+SHIFT+C`] to toggle the browser's `Page Inspector`, and then click on the profile image, which is where we trigger the file selector for the upload form:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>(<code>.jpg,.jpeg,.png</code>) as the allowed file types</p></figcaption></figure>

To get the details of the function `checkFile`, we can go to the browser's `Console` by clicking \[`CTRL+SHIFT+K`], and then we can type the function's name (`checkFile`) to get its details

```javascript
function checkFile(File) {
...SNIP...
    if (extension !== 'jpg' && extension !== 'jpeg' && extension !== 'png') {
        $('#error_message').text("Only images are allowed!");
        File.form.reset();
        $("#submit").attr("disabled", true);
    ...SNIP...
    }
}
```

We can add `PHP` as one of the allowed extensions or modify the function to remove the extension check.

But We can also just remove this function from the HTML code since its primary use appears to be file type validation, and removing it should not break anything.

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can now upload our shell if the verification is on client side and if we did not refresh the page yet since it will not be persistent. If we inspect element and click on our pfp, we should see our web shell ->

```html
<img src="/profile_images/shell.php" class="profile-image" id="profile-image">
```

And then we can go and look at the webshell in the web page:

```
http://SERVER_IP:PORT/profile_images/shell.php?cmd=id
```

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**Try to bypass the client-side file type validations in the above exercise, then upload a web shell to read /flag.txt (try both bypass methods for better practice)**_

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>server side</p></figcaption></figure>

And now on client side ->

We need to erase the check function:

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

`if(validate()){upload()}` becomes `upload()`

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Blacklist Filters

Here we are going to see why using a blacklist of common extensions may not be enough to prevent arbitrary file uploads and discuss several methods to bypass it.

On our new website we tried to intercept an image upload request with Burp, replace the file content and filename with our PHP script's, and forward the request but it did not work

<figure><img src="../../../.gitbook/assets/image (1315).png" alt=""><figcaption></figcaption></figure>

So there is something on the back end retriciting our upload

There are generally two common forms of validating a file extension on the back-end:

1. Testing against a `blacklist` of types
2. Testing against a `whitelist` of types

We can also check the `file type` or the `file content` for type matching. The weakest form of validation amongst these is `testing the file extension against a blacklist of extension`

To bypass a blacklist of web extensions `PayloadsAllTheThings` provides lists of extensions for [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) and [.NET](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP) web applications. We may also use `SecLists` list of common [Web Extensions](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt)

Let's try for php:

<figure><img src="../../../.gitbook/assets/image (1316).png" alt=""><figcaption></figcaption></figure>

And they pretty much all work So if we go back in repeater and try modifying the extension with .phtml ->

<figure><img src="../../../.gitbook/assets/image (1317).png" alt=""><figcaption></figcaption></figure>

And to go and verify ->

```
http://SERVER_IP:PORT/profile_images/shell.phtml?cmd=id
```

<figure><img src="../../../.gitbook/assets/image (1318).png" alt=""><figcaption></figcaption></figure>

_**Try to find an extension that is not blacklisted and can execute PHP code on the web server, and use it to read "/flag.txt"**_

i start by uploading a photo and capturing the request:

<figure><img src="../../../.gitbook/assets/image (1319).png" alt=""><figcaption></figcaption></figure>

I input my webshell, send it to intruder and add the extension for a sniper attack:

<figure><img src="../../../.gitbook/assets/image (1320).png" alt=""><figcaption></figcaption></figure>

I add this little list:[https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst)

<figure><img src="../../../.gitbook/assets/image (1321).png" alt=""><figcaption></figcaption></figure>

And it seems to work pretty well:

<figure><img src="../../../.gitbook/assets/image (1322).png" alt=""><figcaption></figcaption></figure>

We forward one of the successful request:

<figure><img src="../../../.gitbook/assets/image (1323).png" alt=""><figcaption></figcaption></figure>

Or we can&#x20;

## Whitelist Filters

the other type of file extension validation is by utilizing a `whitelist of allowed file extensions`. A whitelist is generally more secure than a blacklist.

Let's say we got the following message:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
