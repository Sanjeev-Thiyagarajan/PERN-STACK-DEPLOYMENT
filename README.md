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

Right now the `ubuntu` user in Postgres does not have a password associated with it. We will need to add a password:

```
ubuntu@ip-172-31-20-1:~$ psql -d postgres
psql (12.4 (Ubuntu 12.4-0ubuntu0.20.04.1))
Type "help" for help.

postgres=# \q
ubuntu@ip-172-31-20-1:~$ psql -d postgres
psql (12.4 (Ubuntu 12.4-0ubuntu0.20.04.1))
Type "help" for help.

postgres=# ALTER USER ubuntu PASSWORD 'password';
ALTER ROLE
postgres=#
```

## 2. Copy github repo to sever

Find a place to store your application code. In this example in the `ubuntu` home directory a new directory called `apps` will be created. Within the new `apps` directory another directory called `yelp-app`. Feel free to store your application code anywhere you see fit

```
cd ~
mkdir apps
cd apps
mkdir yelp-app
```

move inside the `yelp-app` directory and clone the project repo
```
cd yelp-app
git clone https://github.com/Sanjeev-Thiyagarajan/PERN-STACK-DEPLOYMENT.git .
```

## 3. Install Node
To install Node on Ubuntu follow the steps detailed in:
https://github.com/nodesource/distributions/blob/master/README.md

```
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt-get install -y nodejs
```

## 4. Install and Configure PM2
We never want to run *node* directly in production. Instead we want to use a process manager like PM2 to handle running our backend server. PM2 will be responsible for restarting the App if/when it crashes :grin:

```
sudo npm install pm2 -g
```
Point pm2 to the location of the server.js file so it can start the app. We can add the `--name` flag to give the process a descriptive name
```
pm2 start /home/ubuntu/apps/yelp-app/server/server.js --name yelp-app
```

Configure PM2 to automatically startup the process after a reboot

```
ubuntu@ip-172-31-20-1:~$ pm2 startup
[PM2] Init System found: systemd
[PM2] To setup the Startup Script, copy/paste the following command:
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
```
The output above gives you a specific command to run, copy and paste it into the terminal. The command given will be different on your machine depending on the username, so do not copy the output above, instead run the command that is given in your output.

```
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
```

Verify that the App is running

```
pm2 status
```
After verify App is running, save the current list of processes so that the same processes are started during bootup. If the list of processes ever changes in the future, you'll want to do another `pm2 save`

```
pm2 save
```

## 5 Deploy React Frontend
Navigate to the client directory in our App code and run `npm run build`. 

This will create a finalized production ready version of our react frontent in directory called `build`. The build folder is what the NGINX server will be configured to serve.

```
ubuntu@ip-172-31-20-1:~/apps/yelp-app/client$ ls
README.md  build  node_modules  package-lock.json  package.json  public  src
ubuntu@ip-172-31-20-1:~/apps/yelp-app/client$ cd build/
ubuntu@ip-172-31-20-1:~/apps/yelp-app/client/build$ ls
asset-manifest.json  favicon.ico  index.html  logo192.png  logo512.png  manifest.json  precache-manifest.ee13f4c95d9882a5229da70669bb264c.js  robots.txt  service-worker.js  static
ubuntu@ip-172-31-20-1:~/apps/yelp-app/client/build$
```
