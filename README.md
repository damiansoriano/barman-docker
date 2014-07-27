Testing Barman with Docker: barman-docker
=============

To launch the postgres container run:

    sudo docker run -ti --name pg damiansoriano/barman-docker-pg

To launch the backup server run:

    sudo docker run -ti --name backup --link pg:pg damiansoriano/barman-docker-backup

Initial Configuration
--------

After running the containers do the following steps:

* In the pg container run `/etc/init.d/ssh start`, this will start the ssh daemon.
* In the pg container, modify the file `/etc/postgresql/9.3/main/postgresql.conf`, changing backup domain with the IP if the backup container
```
archive_command = 'rsync -a %p barman@backup:/var/lib/barman/main/incoming/%f'
```
* Restart PostgreSQL server with ```service postgresql restart```
* In the backup container run `/etc/init.d/ssh start`, this will start the ssh daemon.
* From the pg container, login as postgres user and try ```ssh barman@backup``` (where backup is the IP of the backup host) to allow the connections from pg to backup.

Initial Backup
--------

At this point barman is ready to receive backups from pg. Run ```barman check main``` to see that the environment is configured correctly.

```
postgres@8cf94e6f82e1:~$ psql
psql (9.3.4)
Type "help" for help.

postgres=# CREATE DATABASE test01;
CREATE DATABASE
postgres=# \c test01;
You are now connected to database "test01" as user "postgres".
test01=# CREATE TABLE table01 (id int);
CREATE TABLE
test01=# INSERT INTO table01 VALUES (1), (2);    
INSERT 0 2
```

From the backup container with the barman user, run:

```
barman@db64ec59f7d3:~$ barman backup main
Starting backup for server main in /var/lib/barman/main/base/20140727T105532
Backup start at xlog location: 0/2000028 (000000010000000000000002, 00000028)
Copying files.
Copy done.
Asking PostgreSQL server to finalize the backup.
Backup end at xlog location: 0/20000B8 (000000010000000000000002, 000000B8)
Backup completed
```

This will create a full backup. You can control de backup with the command ```barman list-backup main```

Incremental Backup
--------

Fron the pg container, create another database with a table and some information within:

```
postgres@8cf94e6f82e1:~$ psql
psql (9.3.4)
Type "help" for help.

postgres=# CREATE DATABASE test02;
CREATE DATABASE
postgres=# \c test02;
You are now connected to database "test02" as user "postgres".
test02=# CREATE TABLE table02 (id int);
CREATE TABLE
test02=# INSERT INTO table02 VALUES (3), (4);
INSERT 0 2
```

After that we should see incomming files in the folder ```/var/lib/barman/main/incoming```, in the backup container. This WAL archives containes information of the creationg of the second database. We can use ```barman cron``` to move them to ```/var/lib/barman/main/wals```.

```
barman@db64ec59f7d3:~/main/wals$ barman list-backup main
main 20140727T105532 - Sun Jul 27 10:55:40 2014 - Size: 24.9 MiB - WAL Size: 16.0 MiB
```

Restoring
--------

At this point we can stop PostgreSQL from pg, to simulate something went wrong with the server.

In the backup container, using the barman user, use the ```barman recover``` to recover the backup in the /var/lib/barman/pg/ folder.

```
barman@db64ec59f7d3:~$ barman list-backup main
main 20140727T105532 - Sun Jul 27 10:55:40 2014 - Size: 24.9 MiB - WAL Size: 16.0 MiB
barman@db64ec59f7d3:~$ mkdir pg
barman@db64ec59f7d3:~$ barman recover main 20140727T105532 /var/lib/barman/pg/
Starting local restore for server main using backup 20140727T105532 
Destination directory: /var/lib/barman/pg/
Copying the base backup.
Copying required wal segments.
The archive_command was set to 'false' to prevent data losses.

Your PostgreSQL server has been successfully prepared for recovery!

Please review network and archive related settings in the PostgreSQL
configuration file before starting the just recovered instance.

WARNING: Before starting up the recovered PostgreSQL server,
please review also the settings of the following configuration
options as they might interfere with your current recovery attempt:

    data_directory = '/var/lib/postgresql/9.3/main'		# use data in another directory
    external_pid_file = '/var/run/postgresql/9.3-main.pid'			# write an extra PID file
    hba_file = '/etc/postgresql/9.3/main/pg_hba.conf'	# host-based authentication file
    ident_file = '/etc/postgresql/9.3/main/pg_ident.conf'	# ident configuration file
    ssl_cert_file = '/etc/ssl/certs/ssl-cert-snakeoil.pem'		# (change requires restart)
    ssl_key_file = '/etc/ssl/private/ssl-cert-snakeoil.key'		# (change requires restart)
```

With the root user in the backup container, restore the structure to proceed with the recovery as follows. You will need to delete the pg_xlog/ folder content since it may contain information that will not be used for the recovery process.

```
root@db64ec59f7d3:/# cd /var/lib/postgresql/9.3/
root@db64ec59f7d3:/var/lib/postgresql/9.3# rm -r main/
root@db64ec59f7d3:/var/lib/postgresql/9.3# cp -r /var/lib/barman/pg/ ./main
root@db64ec59f7d3:/var/lib/postgresql/9.3# cp -r /var/lib/barman/main/wals/ ./wals
root@db64ec59f7d3:/var/lib/postgresql/9.3# echo restore_command = \'cp /var/lib/postgresql/9.3/wals/0000000100000000/%f "%p"\' > main/recovery.conf
root@db64ec59f7d3:/var/lib/postgresql/9.3# rm -r /var/lib/postgresql/9.3/main/pg_xlog/*
root@db64ec59f7d3:/var/lib/postgresql/9.3# chown -R postgres.postgres *
```

Now, if the PostgreSQL service is started it will backup the server taking into account the WAL archives. The backup was sucessfully restored. You can take a look at the PostgreSQL log with the following command:

```
root@db64ec59f7d3:/# tail -n 200 /var/log/postgresql/postgresql-9.3-main.log
```

Miscellaneous
--------

To link containers from pg to backup you should run containers this way:

    sudo docker run -ti --name backup damiansoriano/barman-docker-backup
    sudo docker run -ti --name pg --link backup:backup damiansoriano/barman-docker-pg

