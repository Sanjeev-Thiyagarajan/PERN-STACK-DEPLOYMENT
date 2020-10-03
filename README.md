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
