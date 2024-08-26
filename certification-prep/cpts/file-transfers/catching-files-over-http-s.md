# 🟨 Catching Files over HTTP/S

A good alternative for transferring files to `Apache` is [Nginx](https://www.nginx.com/resources/wiki/)

Apache executes anything ending in `PHP` so uploading web shells and executing them is less complicated. Configuring `Nginx` to use PHP is nowhere near as simple.

**Create a Directory to Handle Uploaded Files**

```shell-session
ElFelixi0@htb[/htb]$ sudo mkdir -p /var/www/uploads/SecretUploadDirectory
```

**Change the Owner to www-data**

```shell-session
ElFelixi0@htb[/htb]$ sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory
```

**Create Nginx Configuration File**

We need to create the file `/etc/nginx/sites-available/upload.conf` with the content:

```shell-session
server {
    listen 9001;
    
    location /SecretUploadDirectory/ {
        root    /var/www/uploads;
        dav_methods PUT;
    }
}
```

**Symlink our Site to the sites-enabled Directory**

```shell-session
ElFelixi0@htb[/htb]$ sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/
```

**Start Nginx**

```shell-session
ElFelixi0@htb[/htb]$ sudo systemctl restart nginx.service
```

We can encounter errors such as port already running, so we need to verify errors ->

```shell-session
ElFelixi0@htb[/htb]$ tail -2 /var/log/nginx/error.log
```

We see there is already a module listening on port 80. To get around this, we can remove the default Nginx configuration, which binds on port 80.

```shell-session
ElFelixi0@htb[/htb]$ sudo rm /etc/nginx/sites-enabled/default
```

Now we can test uploading by using `cURL` to send a `PUT` request. In the below example, we will upload the `/etc/passwd` file to the server and call it users.txt

```shell-session
ElFelixi0@htb[/htb]$ curl -T /etc/passwd http://localhost:9001/SecretUploadDirectory/users.txt
```

```shell-session
ElFelixi0@htb[/htb]$ sudo tail -1 /var/www/uploads/SecretUploadDirectory/users.txt 

user65:x:1000:1000:,,,:/home/user65:/bin/bash
```
