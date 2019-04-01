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
