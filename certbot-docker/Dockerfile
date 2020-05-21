FROM ubuntu:18.04

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get -y upgrade
RUN apt-get install -y software-properties-common
RUN add-apt-repository universe
RUN add-apt-repository ppa:certbot/certbot
RUN apt-get update
RUN apt-get install -y certbot python3-certbot-apache

COPY ./run-certbot /run-certbot
RUN chmod +x /run-certbot

ENTRYPOINT ["/run-certbot"]
