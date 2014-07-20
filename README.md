barman-docker
=============

To launch the postgres container run:

    sudo docker run -ti --name pg damiansoriano/barman-docker-pg

To launch the backup server run:

    sudo docker run -ti --name backup --link pg:pg damiansoriano/barman-docker-backup

Or if we want to connnect the other way arround:

    sudo docker run -ti --name backup damiansoriano/barman-docker-backup
    sudo docker run -ti --name pg --link backup:backup damiansoriano/barman-docker-pg
