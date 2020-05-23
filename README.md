# Docker_Compose_Multi_Container_Tool
[docker-compose documentation](https://docs.docker.com/compose/compose-file/)
### Our containers will often require other containers such as SQL or a key value. And other applications that we'll need to run in containers, like procies or web frontends or backend workers, and so on. What if we had a way to connect all those pieces of our solution together, where we did't need to remember all of our docker rn options, and then get them into discreet, virtual networks with relationships between them, and only expose the public ports, and then spin them all up and tear them down with one command. This is Docker Compose.
![docker_compose_yml](https://github.com/NoriKaneshige/Docker_Compose_Multi_Container_Tool/blob/master/docker_compose_yml.png)
## template of yml file
```
version: '3.1'  # if no version is specified then v1 is assumed. Recommend v2 minimum

services:  # containers. same as docker run
  servicename: # a friendly name. this is also DNS name inside network
    image: # Optional if you use build:
    command: # Optional, replace the default CMD specified by the image
    environment: # Optional, same as -e in docker run
    volumes: # Optional, same as -v in docker run
  servicename2:

volumes: # Optional, same as docker volume create

networks: # Optional, same as docker network create
```
## Let's see the yml file that is doing exactly the same as what docker run command did with jekyll-serve
## docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve
### We specified an image and named as jekyll, gave it a volume. '.' means this is the current directory I'm in and just use this directory that I'm running the compose file from. Then, ports with -p option.
```
version: '2'

# same as 
# docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve

services:
  jekyll:
    image: bretfisher/jekyll-serve
    volumes:
      - .:/site
    ports:
      - '80:4000'
```
## Look at another example, To set up a three-database server cluster behind a Ghost web server with multiple environment variables for each one of the containers.
```
version: '3'

services:
  ghost:
    image: ghost
    ports:
      - "80:2368"
    environment:
      - URL=http://localhost
      - NODE_ENV=production
      - MYSQL_HOST=mysql-primary
      - MYSQL_PASSWORD=mypass
      - MYSQL_DATABASE=ghost
    volumes:
      - ./config.js:/var/lib/ghost/config.js
    depends_on:
      - mysql-primary
      - mysql-secondary
  proxysql:
    image: percona/proxysql
    environment: 
      - CLUSTER_NAME=mycluster
      - CLUSTER_JOIN=mysql-primary,mysql-secondary
      - MYSQL_ROOT_PASSWORD=mypass
   
      - MYSQL_PROXY_USER=proxyuser
      - MYSQL_PROXY_PASSWORD=s3cret
  mysql-primary:
    image: percona/percona-xtradb-cluster:5.7
    environment: 
      - CLUSTER_NAME=mycluster
      - MYSQL_ROOT_PASSWORD=mypass
      - MYSQL_DATABASE=ghost
      - MYSQL_PROXY_USER=proxyuser
      - MYSQL_PROXY_PASSWORD=s3cret
  mysql-secondary:
    image: percona/percona-xtradb-cluster:5.7
    environment: 
      - CLUSTER_NAME=mycluster
      - MYSQL_ROOT_PASSWORD=mypass
   
      - CLUSTER_JOIN=mysql-primary
      - MYSQL_PROXY_USER=proxyuser
      - MYSQL_PROXY_PASSWORD=s3cret
    depends_on:
      - mysql-primary
```
## Trying out basic compouse commands
![docker_compose_cli](https://github.com/NoriKaneshige/Docker_Compose_Multi_Container_Tool/blob/master/docker_compose_cli.png)
### here, we have two containers. The first one is nginx server that is configured as proxy, which is going to be listening on my port 80 on my local machine. Also, there is a volume command where we do a bind mount to a file in this directory called an nginx config file. For examle, this directory is a Git repository that stores these Config files for me. It allows me to map that file into the container.
### docker compouse up command should
### 1) start up both containers
### 2) create a private network for the two containers
### 3) automatically bind mount that file
### 4) open up th port
### 5) and start dumping logs out to my screen.
```
Koitaro@MacBook-Pro-3 compose-sample-2 % ls -alF
total 16
drwxr-xr-x@  4 Koitaro  staff   128 May 14 15:31 ./
drwxr-xr-x  36 Koitaro  staff  1152 May 14 15:38 ../
-rw-r--r--   1 Koitaro  staff   297 May 14 15:31 docker-compose.yml
-rw-r--r--   1 Koitaro  staff   298 May 14 15:31 nginx.conf

# docker-compose.yml
version: '3'

services:
  proxy:
    image: nginx:1.13 # this will use the latest version of 1.13.x
    ports:
      - '80:80' # expose 80 on host and sent to 80 in container
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
  web:
    image: httpd  # this will use httpd:latest
```
