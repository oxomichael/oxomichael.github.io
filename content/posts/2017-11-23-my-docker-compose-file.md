---
categories: docker compose dev php apache
date: "2017-11-23T22:00:00Z"
lang: en
ref: 2017-11-23-my-docker-compose-file
title: My Dev Docker Compose File
---

## Use docker to compose your dev environment
Working with Apache, PHP, MariaDB, ...

Here is my sample compose file

```
version: '2.1'
services:
 # PHP 7.0 FPM
 dev-fpm70:
  container_name: dev-fpm70
  image: oxo/php-fpm:7.0-debian
  volumes:
   - /home/michael/workspace:/home/vhosts
  links:
   - "dev-db:db"
   - "dev-mailcatcher:mailcatcher"
  logging:
   driver: "json-file"
   options:
    max-size: "1g"
    max-file: "10"
  environment:
   LOCAL_USER_ID: 1000
 # MailCatcher
 dev-mailcatcher:
  container_name: dev-mailcatcher
  image: oxo/mailcatcher:0.6.5
  logging:
   driver: "json-file"
   options:
    max-size: "1g"
    max-file: "10"    
 # MariaDB
 dev-db:
  container_name: dev-db
  image: oxo/mariadb:10.2-debian
  user: mysql
  volumes:
   - /home/michael/workspace/database:/var/lib/mysql
  logging:
   driver: "json-file"
   options:
    max-size: "500m"
    max-file: "9"
  environment:
   MARIADB_PASS: password
 # Apache HTTPd
 dev-httpd:  
  container_name: dev-httpd
  image: oxo/httpd:2.4-debian
  volumes:
   - /home/michael/workspace:/home/vhosts
  links:
   - "dev-fpm70:fpm70"
  logging:
   driver: "json-file"
   options:
    max-size: "1g"
    max-file: "10"
  networks:
   default:
   front:
    ipv4_address: 172.10.0.2

networks:
 front:
  driver: bridge
  ipam:
   driver: default
   config:
    - subnet: 172.10.0.0/16
      gateway: 172.10.0.1

```
### How to use it
TODO

### TODO
- Add webpack-dev-server container

## Install Docker
```
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common    
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce
```

## Install Docker Compose
```
curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## My Docker files
See all my docker files : [https://github.com/oxomichael/envdev-dockerfiles](https://github.com/oxomichael/envdev-dockerfiles)

## Monitor
- [Ctop](https://github.com/bcicen/ctop)
