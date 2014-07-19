FROM ubuntu:14.04

MAINTAINER Damian Soriano <ds@ingadhoc.com>

RUN apt-get clean
RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get -y install postgresql

RUN service postgresql start

CMD ["/bin/bash"]
