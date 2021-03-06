---
layout: post
title: Postgres.app with unix socket
---

One of the best ways to get [PostgreSQL](http://www.postgresql.org) running
quickly on your computer is [Postgres.app](http://postgresapp.com). Unfortunately
Postgres.app doesn't enable connections via unix socket by default. I like to use
sockets because they're faster more secure.

The server is configured, by default, to allow connections by your username from
`localhost` on postgres' default port `5432`. If you don't to specify the host
`psql` will attempt to connect via unix socket and fail with an error like:

    $ psql
    psql: could not connect to server: No such file or directory
            Is the server running locally and accepting
            connections on Unix domain socket "/var/pgsql_socket/.s.PGSQL.5432"?

> **Note**: The default socket location for the `psql` command is "/tmp". The
> `psql` command I used during the writing of this article was configured to
> use "/var/pgsql_socket".

## Configure Postgres.app to use unix sockets

The following steps will configure Postgres.app to allow connections via unix
socket for a more flexible experience. _Note_: These instructions are tested
against Mac OS X 10.8.

### Postgres.app v9.2

1. Download and run [Postgres.app](http://postgresapp.com) so that the default
   configuration is initialized in `~/Library/Application Support/Postgres`.
2. Quit Postgres.app (from the Mac OS X menu bar).
3. Open the file `~/Library/Application Support/Postgres/var/postgresql.conf` in
   your favorite text editor.
4. Uncomment the line `#unix_socket_directory = ''` and change it to
   `unix_socket_directory = '/var/pgsql_socket'`.
5. Create the directory `/var/pgsql_socket` if it doesn't exist. (may require `sudo`)
6. Run `chmod 770 /var/pgsql_socket`. (may require `sudo`)
7. Run `chown root:staff /var/pgsql_socket`. (may require `sudo`)

Now you can connect to the server by unix socket!

    $ psql
    psql (9.1.5, server 9.2.2)
    WARNING: psql version 9.1, server version 9.2.
             Some psql features might not work.
    Type "help" for help.

    your_username=#

### v9.3

Postgres.app 9.3 introduces [app sandboxing](https://developer.apple.com/library/mac/documentation/security/conceptual/AppSandboxDesignGuide/AppSandboxInDepth/AppSandboxInDepth.html)
which changes the location for the configuration data to `~/Library/Containers/com.heroku.postgres/Data/Library/Application Support/Postgres/var`.
This is a little confusing, especially considering the lack of documentation on the Postgres.app
site and in the docs. See [this Github issue](https://github.com/PostgresApp/PostgresApp/issues/131)
for more information.

It's also worth noting that from PostgreSQL 9.2 to 9.3 the unix socket configuration
[changed from `unix_socket_directory` to `unix_socket_directories`](http://www.postgresql.org/docs/9.3/static/runtime-config-connection.html),
so make sure your `postgresql.conf` file uses the correct variable name!

### v9.3.1

Aaaaaaand they've taken sandboxing out in 9.3.1. To add to the fun, they've also
renamed the app to `Postgres93.app` and adjusted the Application Support directory
accordingly. Now you can find the config file at:

```
~/Library/Application Support/Postgres93/var/postgresql.conf
```

Remember that you'll probably want to [update your `PATH`](https://github.com/iamvery/dotfiles/commit/94ca807c79f09a7034a689daed0421ab98230ef9#diff-ec20fb240e117fea7b0049c21edf1ef3).

## Configuring `psql` socket path

If your `psql` command is configured to use the "/tmp" directory, you may override this default
with the `PGHOST` environment variable. I include this line in my shell rc file:

   export PGHOST=/var/pgsql_socket
