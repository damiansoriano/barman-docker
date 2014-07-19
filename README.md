barman-docker
=============

To launch the postgres container run:

    sudo docker run -ti --name pg damiansoriano/barman-docker-pg

To launch the backup server run:

    sudo docker run -ti --name backup --link pg:pg damiansoriano/barman-docker-backup
