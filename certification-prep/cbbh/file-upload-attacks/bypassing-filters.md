# 🦘 Bypassing Filters

Many web applications only rely on front-end JavaScript code to validate the selected file format before it is uploaded

we can easily bypass it by directly interacting with the server, skipping the front-end validations altogether. We may also modify the front-end code through our browser's dev tools to disable any validation in place.

Let's say we have an upload image form that blocks our .php file:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### Back-end Request Modification

We start by sending the request to burp:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

The important thing to look at is `filename="HTB.png"` and the file content at the end of the request. If we modify the `filename` to `shell.php` and modify the content to the web shell we used in the previous section; we would be uploading a `PHP` web shell instead of an image.

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption><p><code>File successfully uploaded</code></p></figcaption></figure>

### Disabling Front-end Validation

Let's start by \[`CTRL+SHIFT+C`] to toggle the browser's `Page Inspector`, and then click on the profile image, which is where we trigger the file selector for the upload form:

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption><p>(<code>.jpg,.jpeg,.png</code>) as the allowed file types</p></figcaption></figure>

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

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

We can now upload our shell if the verification is on client side and if we did not refresh the page yet since it will not be persistent. If we inspect element and click on our pfp, we should see our web shell ->

```html
<img src="/profile_images/shell.php" class="profile-image" id="profile-image">
```

And then we can go and look at the webshell in the web page:

```
http://SERVER_IP:PORT/profile_images/shell.php?cmd=id
```

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

_**Try to bypass the client-side file type validations in the above exercise, then upload a web shell to read /flag.txt (try both bypass methods for better practice)**_

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption><p>server side</p></figcaption></figure>

And now on client side ->

We need to erase the check function:

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

`if(validate()){upload()}` becomes `upload()`

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
