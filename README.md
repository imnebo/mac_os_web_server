# mac_os_web_server
this guide will help you trough the necessary steps to configure you mac as a web server for development.

# Installing Brew

[Brew Package Manager](https://brew.sh)

# Installing PHP via Brew
PHP 7.1 it's already installed in your mac so you can skip this step or if you want to use PHP 7.3 install it via brew

In your command-line run the following commands:

`brew install php`

this will install also the necessary PHP dependency

# Installing Postgres via Brew

In your command-line run the following commands:

1. `brew doctor`
1. `brew update`

## Installing
1. In your command-line run the command: `brew install postgres`
2. Install postgis, type in the command-line: `brew install postgis`
2. Start PostgreSQL server: `pg_ctl -D /usr/local/var/postgres start`
3. Create Database: `initdb /usr/local/var/postgres`

    If terminal shows an error

    ```
    initdb: directory "/usr/local/var/postgres" exists but is not empty
    If you want to create a new database system, either remove or empty
    the directory "/usr/local/var/postgres" or run initdb
    with an argument other than "/usr/local/var/postgres".
    ```

    Remove old database file

    `rm -r /usr/local/var/postgres`

    Run the initdb command again

    `initdb /usr/local/var/postgres`

    Create a new database

    `createdb postgis_test`

6. Managing Postgres
    The Homebrew package manager includes launchctl plists to start automatically. For more information run brew info postgres.

    Start manually:

    `pg_ctl -D /usr/local/var/postgres start`

    Stop manually:

    `pg_ctl -D /usr/local/var/postgres stop`

    Start automatically:

    "To have launchd start postgresql now and restart at login:"

    `brew services start postgresql`


## Commands

### Create database
```
createdb <database_name>
```
> `createdb mydb_development`

### List databases
```
psql -U postgres -l
```

### Show tables in database
```
psql -U postgres -d <database_name>
```
> `psql -U postgres -d mydb_development`

### Drop database
```
dropdb <database_name>
```
> `dropdb mydb_development`

### Restart database
```
dropdb <database_name> && createdb <database_name>
```
> `dropdb mydb_development && createdb mydb_development`

# Installing pgAdmin 4

[pgAdmin 4 installer](https://www.pgadmin.org/download/pgadmin-4-macos/)

just download the app and copy it in your application folder.

## Checking Postgres database

Open pgAdmin 4 and create a new server, use your credentials to connect to it (usually it's your mac user without password)

# Configuring Apache

To get started, search for brew PHP `libphp7.so`

`sudo find / -name libphp7.so`

In my case is:

> `/usr/local/Cellar/php/7.3.3/lib/httpd/modules/libphp7.so`

Now edit the Apache configuration file as root:

`sudo nano /etc/apache2/httpd.conf`

### Enable PHP by adding at line 177, after the original PHP lib, the new PHP:

(Original PHP 7.1)
`#LoadModule php7_module libexec/apache2/libphp7.so`

(Brew PHP 7.3)
`LoadModule php7_module /usr/local/Cellar/php/7.3.3/lib/httpd/modules/libphp7.so`

If you want to use the preinstalled PHP 7.1, simply uncomment the module

`LoadModule php7_module libexec/apache2/libphp7.so`


### If you want to run Perl scripts, you will have to do something similar:

Enable Perl by uncommenting line 178, changing:

`#LoadModule perl_module libexec/apache2/mod_perl.so`

to

`LoadModule perl_module libexec/apache2/mod_perl.so`

### Enable personal websites by uncommenting the following at line 174:

`#LoadModule userdir_module libexec/apache2/mod_userdir.so`

to

`LoadModule userdir_module libexec/apache2/mod_userdir.so`

and do the same at line 511:

`#Include /private/etc/apache2/extra/httpd-userdir.conf`

to

`Include /private/etc/apache2/extra/httpd-userdir.conf`

### Enable virtual host configuration

line 168:

`#LoadModule vhost_alias_module libexec/apache2/mod_vhost_alias.so`

to

`LoadModule vhost_alias_module libexec/apache2/mod_vhost_alias.so`

### Enable mod rewrite

line 176

`#LoadModule rewrite_module libexec/apache2/mod_rewrite.so`

to

`LoadModule rewrite_module libexec/apache2/mod_rewrite.so`

### modify the options

add or modify the existing piece of code at line 262

`Options FollowSymLinks Multiviews Indexes`

### include the virtual host configuration

line 520

`#include /private/etc/apache2/extra/httpd-vhosts.conf`

to

`include /private/etc/apache2/extra/httpd-vhosts.conf`

### include User home directories conf

line 514

`#Include /private/etc/apache2/extra/httpd-userdir.conf`

to

`Include /private/etc/apache2/extra/httpd-userdir.conf`

Now save and quit.


### modifying the configuration

Open the userdir conf file you just enabled above with:

`sudo vi /etc/apache2/extra/httpd-userdir.conf`

and uncomment the following at line 16:

`#Include /private/etc/apache2/users/*.conf`

to

`Include /private/etc/apache2/users/*.conf`

Save and exit.

Open the vhosts conf file you just enabled above with:

`sudo vi /private/etc/apache2/extra/httpd-vhosts.conf`

modify the existing code with the below one:

```
<VirtualHost *:80>
        DocumentRoot "/Users/<your short user name>/Sites"
        ServerName <your dummy domain name>
        ErrorLog "/private/var/log/apache2/<your dummy domain name>-error_log"
        CustomLog "/private/var/log/apache2/<your dummy domain name>-access_log" common
        <Directory "/Users/<your short user name>/Sites">
            AllowOverride All
            Require all granted
        </Directory>
</VirtualHost>
```

### Creating the Sites folder

Lion and later versions no longer create personal web sites by default. If you already had a Sites folder in Snow Leopard, it should still be there. To create one manually, enter the following:

`mkdir ~/Sites`

`echo "<html><body><h1>My site works</h1></body></html>" > ~/Sites/index.html`

### Check your .conf file

While you are in `/etc/apache2`, double-check to make sure you have a user config file. It should exist at the path: `/etc/apache2/users/<your short user name>.conf`.

That file may not exist and if you upgrade from an older version, you may still not have it. It does appear to be created when you create a new user. If that file doesn't exist, you will need to create it with:

`sudo vi /etc/apache2/users/<your short user name>.conf`

Use the following as the content:

```
<Directory "/Users/<your short user name>/Sites/">

  AddLanguage en .en

  AddHandler perl-script .pl

  PerlHandler ModPerl::Registry

  Options Indexes MultiViews FollowSymLinks ExecCGI

  AllowOverride None

  Require host localhost

</Directory>
```
### Restart apache

Now you are ready to restart Apache itself. But first, do a sanity check. Sometimes copying and pasting from an internet forum can insert invisible, invalid characters into config files. Check your configuration by running the following command in the Terminal:

`apachectl configtest`

The final step is to restart Apache:

`apachectl restart`

### Mapping the dummy domain

In order to access sites locally you need to edit your hosts file.

`vi /etc/hosts`

Add a line to the bottom of this file for your virtual host. It should match the value you used for the ServerName configuration. For example, my site:

`127.0.0.1       <your dummy domain name>`

I like to run the following to clear the local DNS cache:

`dscacheutil -flushcache`

# Testing

now you can open a browser and type your `domain` or `localhost` and it will return `My site works`

### Now try PHP.

Create a PHP info file with:

`echo "<?php echo phpinfo(); ?>" > ~/Sites/info.php`

And test it by entering the following into the address bar:

http://localhost/info.php

You should see your PHP configuration information.
