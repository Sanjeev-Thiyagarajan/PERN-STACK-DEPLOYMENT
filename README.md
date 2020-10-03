# Deploying PERN stack on Ubuntu 20.04

> Detailed step by step procedure to deploying PERN(Postgres, Express, React, Node) stack on Ubuntu 20.04 with NGINX and SSL

## 1. Install and Configure PostgreSQL

Update packages
```
sudo apt update && sudo apt upgrade -y
```

install PostgreSQL
```
sudo apt install postgresql postgresql-contrib -y
```

PostgreSQL will create an initial user called `postgres`. In addition, PostgreSQL uses what's reffered to as peer authentication for local connections. This means that PostgreSQL obtains the username from the linux kernel and uses that to connect to the database. This requires that any user configured on postgres to have an equivalent user defined on Ubuntu. Postgres installation will have automatically created a `postgres` user on Ubuntu as well to allow local connection. this can be verified by running the command:

```
ubuntu@ip-172-31-20-1:~$ sudo cat /etc/passwd | grep -i postgres
postgres:x:113:120:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
```

To connect to Postgres, switch  to the `postgres` user and run psql:

```
ubuntu@ip-172-31-20-1:~$ sudo -i -u postgres
postgres@ip-172-31-20-1:~$ psql
psql (12.4 (Ubuntu 12.4-0ubuntu0.20.04.1))
Type "help" for help.

postgres=#
```

For this tutorial, the node process process will be run under the `ubuntu` user and for the sake of simplicity, an `ubuntu` user will be created on Postgres as well. If you want to use a different user feel free to create a different user in postgres.

To create a Postgres user run the following command which will give an interactive prompt for configuring the new user. For the sake of simplicity, the ubuntu user will be a superuser, which is the equivalent of being a root user on linux. The super user will have the ability to create/delete/modify databases and users.

```
postgres@ip-172-31-20-1:~$ createuser --interactive
Enter name of role to add: ubuntu
Shall the new role be a superuser? (y/n) y
```

login to postgres using the `postgres` user for now to verify the new `ubuntu` user was created successfully

```
postgres@ip-172-31-20-1:~$ psql
psql (12.4 (Ubuntu 12.4-0ubuntu0.20.04.1))
Type "help" for help.

postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 sanjeev   | Superuser, Create role, Create DB                          | {}
 test      |                                                            | {}
 test1     | Superuser, Create role, Create DB                          | {}
 ubuntu    | Superuser, Create role, Create DB                          | {}

postgres=#
```

Exit out of the psql by running `\q` and also exit out of the `postgres` user by running `exit` on the command line

Let's try to run `psql` as the ubuntu user now. An error similar to the one below should be observed

```
ubuntu@ip-172-31-20-1:~$ psql
psql: error: could not connect to server: FATAL:  database "ubuntu" does not exist
```

The reason for this is that Postgres by default tries to connect to a database that is the same name as the user. Since the user is `ubuntu` it tries to connect to a database called `ubuntu` as well which does not exist. We can go in and create a database called `ubuntu` so that it will automtically connect, however I find this unnecessry. Instead we can pass in the `-d` flag and connect to a database that we know exists like the `postgres` 

```
psql -d postgres
```
