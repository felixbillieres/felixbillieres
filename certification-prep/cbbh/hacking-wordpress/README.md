# 💥 Hacking WordPress

[WordPress](https://wordpress.org/) is the most popular open source Content Management System (CMS), A CMS is a powerful tool that helps build a website without the need to code everything from scratch

WordPress requires a fully installed and configured LAMP stack (Linux operating system, Apache HTTP Server, MySQL database, and the PHP programming language) before installation on a Linux host. After installation, all WordPress supporting files and directories will be accessible in the webroot located at `/var/www/html`.

Here is the basic file structure of a default WP install:

```shell-session
ElFelixi0@htb[/htb]$ tree -L 1 /var/www/html
.
├── index.php
├── license.txt
├── readme.html
├── wp-activate.php
├── wp-admin
├── wp-blog-header.php
├── wp-comments-post.php
├── wp-config.php
├── wp-config-sample.php
├── wp-content
├── wp-cron.php
├── wp-includes
├── wp-links-opml.php
├── wp-load.php
├── wp-login.php
├── wp-mail.php
├── wp-settings.php
├── wp-signup.php
├── wp-trackback.php
└── xmlrpc.php
```

And here are some key files:

* `index.php` is the homepage of WordPress.
* `license.txt` contains useful information such as the version WordPress installed.
* `wp-activate.php` is used for the email activation process when setting up a new WordPress site.
* `wp-admin` folder contains the login page for administrator access and the backend dashboard. Once a user has logged in, they can make changes to the site based on their assigned permissions. The login page can be located at one of the following paths:
  * `/wp-admin/login.php`
  * `/wp-admin/wp-login.php`
  * `/login.php`
  * `/wp-login.php`
* `wp-config.php` file contains information required by WordPress to connect to the database, such as the database name, database host, username and password, authentication keys and salts, and the database table prefix.
* `wp-content` folder is the main directory where plugins and themes are stored.

And here are the 5 types of users for a standard WP install:

| Role          | Description                                                                                                                                            |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Administrator | This user has access to administrative features within the website. This includes adding and deleting users and posts, as well as editing source code. |
| Editor        | An editor can publish and manage posts, including the posts of other users.                                                                            |
| Author        | Authors can publish and manage their own posts.                                                                                                        |
| Contributor   | These users can write and manage their own posts but cannot publish them.                                                                              |
| Subscriber    | These are normal users who can browse posts and edit their profiles.                                                                                   |
