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
![docker_compose_up](https://github.com/NoriKaneshige/Docker_Compose_Multi_Container_Tool/blob/master/docker_compose_up.png)
### This is actually the Apache server replying, not the Nginx server. The traffic is going through the Nginx reverse proxy. It's repeating the traffic over to the Apache server. The Apache server is responding with its default basic HTML file because we did not change anything. Then, Nginx is repeating it back to me.
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
    


Koitaro@MacBook-Pro-3 compose-sample-2 % docker-compose up
Creating network "compose-sample-2_default" with the default driver
Pulling proxy (nginx:1.13)...
1.13: Pulling from library/nginx
f2aa67a397c4: Downloading [>                                                  ]  234.6kB/22.5MBulling fs layer
f2aa67a397c4: Downloading [=>                                                 ]    464kB/22.5MBownloading [==================================================>] f2aa67a397c4: Pull complete
3c091c23e29d: Pull complete
4a99993b8636: Pull complete
Digest: sha256:b1d09e9718890e6ebbbd2bc319ef1611559e30ce1b6f56b2e3b479d9da51dc35
Status: Downloaded newer image for nginx:1.13
Pulling web (httpd:)...
latest: Pulling from library/httpd
afb6ec6fdc1c: Already exists
5a6b409207a3: Pull complete
41e5e22239e2: Pull complete
9829f70a6a6b: Pull complete
3cd774fea202: Pull complete
Digest: sha256:db9c3bca36edb5d961d70f83b13e65e552641e00a7eb80bf435cbe9912afcb1f
Status: Downloaded newer image for httpd:latest
Creating compose-sample-2_web_1   ... done
Creating compose-sample-2_proxy_1 ... done
Attaching to compose-sample-2_web_1, compose-sample-2_proxy_1
web_1    | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.18.0.2. Set the 'ServerName' directive globally to suppress this message
web_1    | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.18.0.2. Set the 'ServerName' directive globally to suppress this message
web_1    | [Sat May 23 19:24:06.937947 2020] [mpm_event:notice] [pid 1:tid 139932259239040] AH00489: Apache/2.4.43 (Unix) configured -- resuming normal operations
web_1    | [Sat May 23 19:24:06.948220 2020] [core:notice] [pid 1:tid 139932259239040] AH00094: Command line: 'httpd -D FOREGROUND'
proxy_1  | 172.18.0.1 - - [23/May/2020:19:26:36 +0000] "GET / HTTP/1.1" 200 45 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36" "-"
web_1    | 172.18.0.3 - - [23/May/2020:19:26:36 +0000] "GET / HTTP/1.0" 200 45
proxy_1  | 172.18.0.1 - - [23/May/2020:19:26:36 +0000] "GET /favicon.ico HTTP/1.1" 404 196 "http://localhost/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36" "-"
web_1    | 172.18.0.3 - - [23/May/2020:19:26:36 +0000] "GET /favicon.ico HTTP/1.0" 404 196

web_1    | 172.18.0.3 - - [23/May/2020:20:01:12 +0000] "GET / HTTP/1.0" 304 -
proxy_1  | 172.18.0.1 - - [23/May/2020:20:01:12 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36" "-"
```
## Let's stop it with ctr C and then run it in the background
## Also look at logs to see the same output
```
Gracefully stopping... (press Ctrl+C again to force)
Stopping compose-sample-2_proxy_1 ... done
Stopping compose-sample-2_web_1   ... done

Koitaro@MacBook-Pro-3 compose-sample-2 % docker-compose up -d
Starting compose-sample-2_proxy_1 ... done
Starting compose-sample-2_web_1   ... done
```
## docker-compose --help
```
Koitaro@MacBook-Pro-3 compose-sample-2 % docker-compose --help
Define and run multi-container applications with Docker.

Usage:
  docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             Specify an alternate compose file
                              (default: docker-compose.yml)
  -p, --project-name NAME     Specify an alternate project name
                              (default: directory name)
  --verbose                   Show more output
  --log-level LEVEL           Set log level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  --no-ansi                   Do not print ANSI control characters
  -v, --version               Print version and exit
  -H, --host HOST             Daemon socket to connect to

  --tls                       Use TLS; implied by --tlsverify
  --tlscacert CA_PATH         Trust certs signed only by this CA
  --tlscert CLIENT_CERT_PATH  Path to TLS certificate file
  --tlskey TLS_KEY_PATH       Path to TLS key file
  --tlsverify                 Use TLS and verify the remote
  --skip-hostname-check       Don't check the daemon's hostname against the
                              name specified in the client certificate
  --project-directory PATH    Specify an alternate working directory
                              (default: the path of the Compose file)
  --compatibility             If set, Compose will attempt to convert keys
                              in v3 files to their non-Swarm equivalent
  --env-file PATH             Specify an alternate environment file

Commands:
  build              Build or rebuild services
  config             Validate and view the Compose file
  create             Create services
  down               Stop and remove containers, networks, images, and volumes
  events             Receive real time events from containers
  exec               Execute a command in a running container
  help               Get help on a command
  images             List images
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pull service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  top                Display the running processes
  unpause            Unpause services
  up                 Create and start containers
  version            Show the Docker-Compose version information
```
## Let's list both containers running
```
Koitaro@MacBook-Pro-3 compose-sample-2 % docker-compose ps
          Name                   Command          State         Ports
----------------------------------------------------------------------------
compose-sample-2_proxy_1   nginx -g daemon off;   Up      0.0.0.0:80->80/tcp
compose-sample-2_web_1     httpd-foreground       Up      80/tcp
```
## Let's list all of the services running inside of them
```
Koitaro@MacBook-Pro-3 compose-sample-2 % docker-compose top
compose-sample-2_proxy_1
PID    USER   TIME                    COMMAND
---------------------------------------------------------------
2231   root   0:00   nginx: master process nginx -g daemon off;
2421   101    0:00   nginx: worker process

compose-sample-2_web_1
PID    USER   TIME        COMMAND
---------------------------------------
2218   root   0:00   httpd -DFOREGROUND
2331   bin    0:00   httpd -DFOREGROUND
2332   bin    0:00   httpd -DFOREGROUND
2333   bin    0:00   httpd -DFOREGROUND
```
## Stop containers by docker-compose down
```
Koitaro@MacBook-Pro-3 compose-sample-2 % docker-compose down
Stopping compose-sample-2_proxy_1 ... done
Stopping compose-sample-2_web_1   ... done
Removing compose-sample-2_proxy_1 ... done
Removing compose-sample-2_web_1   ... done
Removing network compose-sample-2_default

Koitaro@MacBook-Pro-3 compose-sample-2 % docker-compose ps
Name   Command   State   Ports
------------------------------
```
