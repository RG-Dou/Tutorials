FROM ubuntu:14.04

RUN apt-get update

RUN apt-get install -y default-jdk

ENV JAVA_HOME=/usr/lib/jvm/default-java

RUN apt-get install -y git

RUN apt-get install -y maven

RUN git clone git://git.apache.org/samza-hello-samza.git hello-samza

RUN apt-get install -y curl
