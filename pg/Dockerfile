FROM ubuntu:14.04

MAINTAINER Damian Soriano <ds@ingadhoc.com>

RUN apt-get clean
RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get -y install postgresql

RUN apt-get -y install python
RUN apt-get -y install python-pip

RUN apt-get -y install rsync

RUN apt-get -y install postgresql-server-dev-all

RUN apt-get -y install libpq-dev python-dev

RUN pip install argcomplete
RUN pip install argh
RUN pip install psycopg2
RUN pip install python-dateutil
RUN pip install distribute

RUN apt-get -y install barman

ADD backup_id_rsa.pub /var/lib/postgresql/.ssh/id_rsa.pub
ADD authorized_keys /var/lib/postgresql/.ssh/authorized_keys

RUN chown -R postgres.postgres /var/lib/postgresql/.ssh

# From http://stackoverflow.com/questions/18173889/cannot-access-centos-sshd-on-docker
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config
RUN sed -ri 's/#UsePAM no/UsePAM no/g' /etc/ssh/sshd_config

RUN /etc/init.d/ssh start

#RUN service postgresql start

EXPOSE 22

CMD ["/bin/bash"]
