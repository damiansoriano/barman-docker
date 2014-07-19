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

#RUN service postgresql start

CMD ["/bin/bash"]
