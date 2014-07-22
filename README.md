barman-docker
=============

To launch the postgres container run:

    sudo docker run -ti --name pg damiansoriano/barman-docker-pg

To launch the backup server run:

    sudo docker run -ti --name backup --link pg:pg damiansoriano/barman-docker-backup

After running the containers do the following steps:

* In the pg container run `/etc/init.d/ssh start`
* In the pg container, modify the file `/etc/postgresql/9.3/main/postgresql.conf`, changing backup domain with the IP if the backup container
```
archive_command = 'rsync -a %p barman@backup:/var/lib/barman/main/incoming/%f'
```
* Restart PostgreSQL server with ```service postgresql restart```
* In the backup container run `/etc/init.d/ssh start`
* From the pg container, login as postgres user and try ```ssh barman@backup``` (where backup is the IP of the backup host) to allow the connections from pg to backup.

To link containers from pg to backup you should run containers this way:

    sudo docker run -ti --name backup damiansoriano/barman-docker-backup
    sudo docker run -ti --name pg --link backup:backup damiansoriano/barman-docker-pg

