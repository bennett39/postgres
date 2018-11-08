# Launching your first server using PostgreSQL on Ubuntu 16.04 - a guide from a novice, for beginners

I had a difficult time launching my very first instance of PostgreSQL on my local machine. Hopefully, this post can help other newbies in my position.

Prior to PostgreSQL, I had experience with SQLite databases. Those are simpler because you don't have to run a server in order to use them. I also used SQLite inside an IDE and everything was set up for me on my first database-based projects.

Long term, though, I know I'll need to be able to use Postgres in production, so I might as well start using it in development. 

With Postgres, you have to run a server that handles database requests. It's not the same as your web server that routes web pages. It's a dedicated Postgres server that makes your databases available to your web application.

This guide will walk you through setting up a local PostgreSQL server on your Linux (Ubuntu 16.04, specifically) computer. I'm mostly writing this guide so that I have these steps to refer back to in the future. This is the guide I wish I could have found on the internet, instead of spending hours troubleshooting each problem myself.

## A note about my shortcomings

I'm fairly new to using bash on the command line. It's entirely possible I've written a wrong or inefficient command below. If you're more experienced than me and want to correct my work, please do. By all means, open a pull request!!

## Step 1. Update your system

Check to make sure your environment is up to date with:

```bash
sudo apt-get update && sudo apt-get upgrade 
```

### Troubleshooting 1.1

At this point, I already had an issue. You might not, but I did. When I tried to update my system I got the following error:

``` 
An error occurred during the signature verification. The repository is not updated and the previous index files will be used. GPG error: http://apt.postgresql.org/pub/repos/apt xenial-pgdg InRelease: The following signatures couldn't be verified because the public key is not available: NOPUBKEY 7FCC7D46ACCC4CF8
```

This error probably occurred because I installed PostgreSQL incorrectly the first time (don't ask...) and had to remove it. The solution was to get a new public key from the Ubuntu key server:

```bash
sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 7FCC7D46ACCC4CF8
```

## Step 2. Install PostgreSQL via apt-get

This applies to users installing on Ubuntu/Debian Linux. If you're using some other distro, you'll probably want to find another guide.

Following the PostgreSQL docs and several guides I found online, I installed PostgreSQL and PostgreSQL-contrib via apt-get:

```bash
sudo apt-get install postgresql postgresql-contrib
``` 

## Step 3. Meet the postgres user on your computer 

As part of the installation, PostgreSQL makes itself at home on your computer and creates itself a user. By default, this user is called `postgres`.

When you want to do something in Postgres, you'll probably want to switch to that user on your computer, since `postgres` owns all the files that the install script created.

The `postgres` user is the one in charge of everything Postgres on your machine. So, switch to that user:

```bash
sudo -i -u postgres
```

## Step 4. If you're lucky, start psql

So, most tutorials on the internet make it seem like you're home free at this point. They say, just log in as the `postgres` user and then type:

```bash
psql
```

And you'll get an interactive screen and a prompt for issuing SQL commands to your PostgreSQL database. If that happens for you, amazing!

That's not what happened for me.

### Troubleshooting 4.1.

I got an error when I typed `psql` for the first time:

```
psql: could not connect to server: No such file or directory Is the server running locally and accepting connections on Unix domain socket "/var/run/postgresql/.s.PGSQL.5432"?
```

The issue is kind of obvious - the server for PostgreSQL isn't running yet! Sure, I installed the package and logged in as `postgres`, but I didn't boot up the postgres server at all. Unless some program did it for me automatically, nothing is running yet for `psql` to connect to.

Now, I needed a command to boot the server. First, I found `pg_ctl`. And that is indeed the command to start a server. I even read the whole man page for it to make sure I understood how it worked. But when I typed it into the command line I was told that command didn't exist.

Stack Overflow to the rescue! A commenter on a post about `pg_ctl` mentions that you should really use `pg_ctlcluster` to start a server, as it wraps the `pg_ctl` command. From the man page:

> `pg_ctlcluster` determines the cluster version and data path and calls the right version of `pg_ctl` with appropriate configuration parameters and paths.

Read the rest of the man page and you'll find the correct syntax for starting a local server:

```bash
pg_ctlcluster [version] [name] [action]
```

In our case, we want to start a version 11 server and name it main, so:

```bash
pg_ctlcluster 11 main start
```

### Troubleshooting 4.2

Another issue I encountered when starting up my server is file permissions problems for the postgres Linux user. Specifically, `postgres` needed to create a directory in my `/var/run` folder. That folder was owned and restricted to the `root` user.

The easiest way to fix this, and the way I used, was to sudo in as root:

```bash
sudo -i
```

And update the permissions for the specific directory that `postgres` needed access to.

There are two ways to do this:

1. You can change the overall permissions for that directory (logged in as root):

```bash
chmod -R 775 [<--or another permission scheme] /var/run/postgresql
```

2. Change the owner of the `postgresql` file to `postgres`:

```bash
chown -R postgres:postgres /var/run/postgresql
```

Option 2 is the safer one since it only gives access to the postgres user.


## Step 5. Test it out

If all goes well (and believe me, I know how much room there is for it to go poorly), you'll have started a PostgreSQL server locally on your Ubuntu machine that you can then access using the `postgres` username by issuing the command `psql`.

Jeez, what a mouthful.

So, does it work? Let's try it:

```bash
(xenial)postgres@localhost:~$ psql -l
                              List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   
-----------+----------+----------+---------+---------+-----------------------
 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
(3 rows)
```

**You have no idea how happy I was when I finally saw that ^^^ table after hours of troubleshooting.**

## Conclusion

You can now use Postgres to do all kinds of stuff. Your machine should now be set up to play nice with any PostgreSQL tutorial you find online. Happy querying!!
