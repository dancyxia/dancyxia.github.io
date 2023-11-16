---
layout: post
title: enable wordpress url redirct(avoid 404 error when accessing the post page)
category: wordpress-dev
tags: [linux wordpress wp]
---

- In order to use mod_rewrite you can type the following command in the terminal:

a2enmod rewrite

- Restart apache2 after

/etc/init.d/apache2 restart
or
service apache2 restart

- Then, if you'd like, you can use the following .htaccess file.

<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule . /index.php [L]
</IfModule>
The above .htaccess file (if placed in your DocumentRoot) will redirect all traffic to an index.php file in the DocumentRoot unless the file exists.

- If non of the above works try editing /etc/apache2/sites-enabled/000-default

almost at the top you will find

<Directory /var/www/>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride None
    Order allow,deny
    allow from all
</Directory>

Change the AllowOverride None to AllowOverride All

=====================================================
### Reference 1: [how to enable mod-rewrite for apache 2.2](http://stackoverflow.com/questions/869092/how-to-enable-mod-rewrite-for-apache-2-2/11846519#11846519)
### Reference 2: [how to install wordpress on LAMP](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-lamp-on-ubuntu-16-04)