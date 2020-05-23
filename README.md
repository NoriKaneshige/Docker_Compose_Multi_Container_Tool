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
## Build a compose file for multi-container searvice
![build_a_compose_file](https://github.com/NoriKaneshige/Docker_Compose_Multi_Container_Tool/blob/master/build_a_compose_file.png)
## docker-compose.yml file
```
version: '2'
# NOTE: move this answer file up a directory so it'll work

services:
  # we use drupal, so I call this drupal
  drupal:
    image: custom-drupal
    build: .
    ports:
      # open up 8080 on my machine, and use port 80 in the container
      # to check which port that container is listening on
      # I could do docker pull drupal and do docker image inspect to see exposed ports
      - "8080:80"
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles       
      - drupal-sites:/var/www/html/sites      
      - drupal-themes:/var/www/html/themes
 
  postgres:
    image: postgres:12.1
    # we don't need any ports since we use the default port inside the network
    # both going to be in the same Docker network, so I don't need for them to connect
    # they just work over that network

    # in postgres, we need to set up a password
    environment:
      - POSTGRES_PASSWORD=mypasswd
    volumes:
      - drupal-data:/var/lib/postgresql/data

# we can use named volumes without anything else in them
volumes:
  drupal-data:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:

```
## Dockerfile
```
FROM drupal:8.8.2


RUN apt-get update && apt-get install -y git \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html/themes

RUN git clone --branch 8.x-3.x --single-branch --depth 1 https://git.drupal.org/project/bootstrap.git \
    && chown -R www-data:www-data bootstrap

WORKDIR /var/www/html
```
## Let's do docker-compose up
## Go to localhost:8080
![drupal_working_locally](https://github.com/NoriKaneshige/Docker_Compose_Multi_Container_Tool/blob/master/drupal_working_locally.png)
```
Koitaro@MacBook-Pro-3 answer % docker-compose up
Building drupal
Step 1/5 : FROM drupal:8.8.2
8.8.2: Pulling from library/drupal
6d28e14ab8c8: Pull complete
9ecd958eae23: Pull complete
4611cd46d612: Pull complete
ad4a2514121d: Pull complete
0b5e20585113: Pull complete
41449f0f405d: Pull complete
8ac46b57a971: Pull complete
fbb79e9a209f: Pull complete
2bbaf0b0b4e6: Pull complete
f2eb2245bea2: Pull complete
341ead679d30: Pull complete
f57261f4f556: Pull complete
36f9693fb790: Pull complete
275c43c6b7cd: Pull complete
a3cddf097b5e: Pull complete
fad623710654: Pull complete
Digest: sha256:a98fc1f914fdbfa68ec229a4f47e8d278b2cca9ed81cad488d742ffb69133131
Status: Downloaded newer image for drupal:8.8.2
 ---> 090e3f252532
Step 2/5 : RUN apt-get update && apt-get install -y git     && rm -rf /var/lib/apt/lists/*
 ---> Running in 403da0cbcbc0
Ign:1 http://deb.debian.org/debian stretch InRelease
Get:2 http://deb.debian.org/debian stretch-updates InRelease [91.0 kB]
Get:3 http://deb.debian.org/debian buster InRelease [121 kB]
Get:4 http://deb.debian.org/debian buster-updates InRelease [49.3 kB]
Get:5 http://deb.debian.org/debian stretch Release [118 kB]
Get:6 http://deb.debian.org/debian stretch Release.gpg [2410 B]
Get:7 http://deb.debian.org/debian stretch-updates/main amd64 Packages [27.9 kB]
Get:8 http://deb.debian.org/debian buster/main amd64 Packages [7905 kB]
Get:9 http://deb.debian.org/debian buster-updates/main amd64 Packages [7380 B]
Get:10 http://deb.debian.org/debian stretch/main amd64 Packages [7083 kB]
Err:11 http://security.debian.org/debian-security stretch/updates InRelease
  Temporary failure resolving 'security.debian.org'
Get:12 http://security.debian.org/debian-security buster/updates InRelease [65.4 kB]
Get:13 http://security.debian.org/debian-security buster/updates/main amd64 Packages [201 kB]
Fetched 15.7 MB in 31s (497 kB/s)
Reading package lists...
W: Failed to fetch http://security.debian.org/debian-security/dists/stretch/updates/InRelease  Temporary failure resolving 'security.debian.org'
W: Some index files failed to download. They have been ignored, or old ones used instead.
Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed:
  git-man less libcurl3-gnutls liberror-perl libpopt0 libx11-6 libx11-data
  libxau6 libxcb1 libxdmcp6 libxext6 libxmuu1 openssh-client rsync xauth
Suggested packages:
  gettext-base git-daemon-run | git-daemon-sysvinit git-doc git-el git-email
  git-gui gitk gitweb git-arch git-cvs git-mediawiki git-svn keychain
  libpam-ssh monkeysphere ssh-askpass openssh-server
The following NEW packages will be installed:
  git git-man less libcurl3-gnutls liberror-perl libpopt0 libx11-6 libx11-data
  libxau6 libxcb1 libxdmcp6 libxext6 libxmuu1 openssh-client rsync xauth
0 upgraded, 16 newly installed, 0 to remove and 1 not upgraded.
Need to get 8590 kB of archives.
After this operation, 41.1 MB of additional disk space will be used.
Get:1 http://deb.debian.org/debian stretch/main amd64 libcurl3-gnutls amd64 7.52.1-5+deb9u9 [290 kB]
Get:2 http://deb.debian.org/debian stretch/main amd64 liberror-perl all 0.17024-1 [26.9 kB]
Get:3 http://deb.debian.org/debian stretch/main amd64 git-man all 1:2.11.0-3+deb9u5 [1433 kB]
Get:4 http://deb.debian.org/debian stretch/main amd64 git amd64 1:2.11.0-3+deb9u5 [4161 kB]
Get:5 http://deb.debian.org/debian stretch/main amd64 libxau6 amd64 1:1.0.8-1 [20.7 kB]
Get:6 http://deb.debian.org/debian stretch/main amd64 libpopt0 amd64 1.16-10+b2 [49.4 kB]
Get:7 http://deb.debian.org/debian stretch/main amd64 less amd64 481-2.1 [126 kB]
Get:8 http://deb.debian.org/debian stretch/main amd64 openssh-client amd64 1:7.4p1-10+deb9u7 [780 kB]
Get:9 http://deb.debian.org/debian stretch/main amd64 libxdmcp6 amd64 1:1.1.2-3 [26.3 kB]
Get:10 http://deb.debian.org/debian stretch/main amd64 libxcb1 amd64 1.12-1 [133 kB]
Get:11 http://deb.debian.org/debian stretch/main amd64 libx11-data all 2:1.6.4-3+deb9u1 [287 kB]
Get:12 http://deb.debian.org/debian stretch/main amd64 libx11-6 amd64 2:1.6.4-3+deb9u1 [748 kB]
Get:13 http://deb.debian.org/debian stretch/main amd64 libxext6 amd64 2:1.3.3-1+b2 [52.5 kB]
Get:14 http://deb.debian.org/debian stretch/main amd64 libxmuu1 amd64 2:1.1.2-2 [23.5 kB]
Get:15 http://deb.debian.org/debian stretch/main amd64 rsync amd64 3.1.2-1+deb9u2 [393 kB]
Get:16 http://deb.debian.org/debian stretch/main amd64 xauth amd64 1:1.0.9-1+b2 [39.6 kB]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 8590 kB in 3s (2802 kB/s)
Selecting previously unselected package libcurl3-gnutls:amd64.
(Reading database ... 13173 files and directories currently installed.)
Preparing to unpack .../00-libcurl3-gnutls_7.52.1-5+deb9u9_amd64.deb ...
Unpacking libcurl3-gnutls:amd64 (7.52.1-5+deb9u9) ...
Selecting previously unselected package liberror-perl.
Preparing to unpack .../01-liberror-perl_0.17024-1_all.deb ...
Unpacking liberror-perl (0.17024-1) ...
Selecting previously unselected package git-man.
Preparing to unpack .../02-git-man_1%3a2.11.0-3+deb9u5_all.deb ...
Unpacking git-man (1:2.11.0-3+deb9u5) ...
Selecting previously unselected package git.
Preparing to unpack .../03-git_1%3a2.11.0-3+deb9u5_amd64.deb ...
Unpacking git (1:2.11.0-3+deb9u5) ...
Selecting previously unselected package libxau6:amd64.
Preparing to unpack .../04-libxau6_1%3a1.0.8-1_amd64.deb ...
Unpacking libxau6:amd64 (1:1.0.8-1) ...
Selecting previously unselected package libpopt0:amd64.
Preparing to unpack .../05-libpopt0_1.16-10+b2_amd64.deb ...
Unpacking libpopt0:amd64 (1.16-10+b2) ...
Selecting previously unselected package less.
Preparing to unpack .../06-less_481-2.1_amd64.deb ...
Unpacking less (481-2.1) ...
Selecting previously unselected package openssh-client.
Preparing to unpack .../07-openssh-client_1%3a7.4p1-10+deb9u7_amd64.deb ...
Unpacking openssh-client (1:7.4p1-10+deb9u7) ...
Selecting previously unselected package libxdmcp6:amd64.
Preparing to unpack .../08-libxdmcp6_1%3a1.1.2-3_amd64.deb ...
Unpacking libxdmcp6:amd64 (1:1.1.2-3) ...
Selecting previously unselected package libxcb1:amd64.
Preparing to unpack .../09-libxcb1_1.12-1_amd64.deb ...
Unpacking libxcb1:amd64 (1.12-1) ...
Selecting previously unselected package libx11-data.
Preparing to unpack .../10-libx11-data_2%3a1.6.4-3+deb9u1_all.deb ...
Unpacking libx11-data (2:1.6.4-3+deb9u1) ...
Selecting previously unselected package libx11-6:amd64.
Preparing to unpack .../11-libx11-6_2%3a1.6.4-3+deb9u1_amd64.deb ...
Unpacking libx11-6:amd64 (2:1.6.4-3+deb9u1) ...
Selecting previously unselected package libxext6:amd64.
Preparing to unpack .../12-libxext6_2%3a1.3.3-1+b2_amd64.deb ...
Unpacking libxext6:amd64 (2:1.3.3-1+b2) ...
Selecting previously unselected package libxmuu1:amd64.
Preparing to unpack .../13-libxmuu1_2%3a1.1.2-2_amd64.deb ...
Unpacking libxmuu1:amd64 (2:1.1.2-2) ...
Selecting previously unselected package rsync.
Preparing to unpack .../14-rsync_3.1.2-1+deb9u2_amd64.deb ...
Unpacking rsync (3.1.2-1+deb9u2) ...
Selecting previously unselected package xauth.
Preparing to unpack .../15-xauth_1%3a1.0.9-1+b2_amd64.deb ...
Unpacking xauth (1:1.0.9-1+b2) ...
Setting up git-man (1:2.11.0-3+deb9u5) ...
Setting up libpopt0:amd64 (1.16-10+b2) ...
Setting up less (481-2.1) ...
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
Processing triggers for mime-support (3.60) ...
Setting up liberror-perl (0.17024-1) ...
Setting up libcurl3-gnutls:amd64 (7.52.1-5+deb9u9) ...
Setting up rsync (3.1.2-1+deb9u2) ...
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of restart.
Processing triggers for libc-bin (2.24-11+deb9u4) ...
Setting up libxdmcp6:amd64 (1:1.1.2-3) ...
Setting up openssh-client (1:7.4p1-10+deb9u7) ...
Setting up git (1:2.11.0-3+deb9u5) ...
Setting up libx11-data (2:1.6.4-3+deb9u1) ...
Setting up libxau6:amd64 (1:1.0.8-1) ...
Setting up libxcb1:amd64 (1.12-1) ...
Setting up libx11-6:amd64 (2:1.6.4-3+deb9u1) ...
Setting up libxmuu1:amd64 (2:1.1.2-2) ...
Setting up libxext6:amd64 (2:1.3.3-1+b2) ...
Setting up xauth (1:1.0.9-1+b2) ...
Processing triggers for libc-bin (2.24-11+deb9u4) ...
Removing intermediate container 403da0cbcbc0
 ---> 6734e7e56107
Step 3/5 : WORKDIR /var/www/html/themes
 ---> Running in d582e6521479
Removing intermediate container d582e6521479
 ---> b85f1bcecefd
Step 4/5 : RUN git clone --branch 8.x-3.x --single-branch --depth 1 https://git.drupal.org/project/bootstrap.git     && chown -R www-data:www-data bootstrap
 ---> Running in d4ef0d16c798
Cloning into 'bootstrap'...
Removing intermediate container d4ef0d16c798
 ---> dfa6a3496c6a
Step 5/5 : WORKDIR /var/www/html
 ---> Running in f8c018403c15
Removing intermediate container f8c018403c15
 ---> bfbc2ee66a92
Successfully built bfbc2ee66a92
Successfully tagged custom-drupal:latest
WARNING: Image for service drupal was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Pulling postgres (postgres:12.1)...
12.1: Pulling from library/postgres
bc51dd8edc1b: Pull complete
d2b355dbb6c6: Pull complete
d237363a1a91: Pull complete
ff4b9d2fde66: Pull complete
646492d166e7: Pull complete
50eeac6fd5fb: Pull complete
502963de6da8: Pull complete
d7263f7627b9: Pull complete
46b135c7e429: Pull complete
259a29a883ed: Pull complete
b3b8f133c3f4: Pull complete
49e91678bd48: Pull complete
15326bd3db00: Pull complete
0aab6409ca4d: Pull complete
Digest: sha256:5181eccc7c903e4f1beffa89a735cb7ed72e0c81d6c34c471552c3fa8bff0858
Status: Downloaded newer image for postgres:12.1
Creating answer_postgres_1 ... done
Creating answer_drupal_1   ... done
Attaching to answer_postgres_1, answer_drupal_1
postgres_1  | The files belonging to this database system will be owned by user "postgres".
postgres_1  | This user must also own the server process.
postgres_1  |
postgres_1  | The database cluster will be initialized with locale "en_US.utf8".
postgres_1  | The default database encoding has accordingly been set to "UTF8".
postgres_1  | The default text search configuration will be set to "english".
postgres_1  |
postgres_1  | Data page checksums are disabled.
postgres_1  |
postgres_1  | fixing permissions on existing directory /var/lib/postgresql/data ... ok
postgres_1  | creating subdirectories ... ok
postgres_1  | selecting dynamic shared memory implementation ... posix
postgres_1  | selecting default max_connections ... 100
postgres_1  | selecting default shared_buffers ... 128MB
postgres_1  | selecting default time zone ... Etc/UTC
postgres_1  | creating configuration files ... ok
drupal_1    | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.20.0.3. Set the 'ServerName' directive globally to suppress this message
drupal_1    | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.20.0.3. Set the 'ServerName' directive globally to suppress this message
drupal_1    | [Sat May 23 21:00:35.997859 2020] [mpm_prefork:notice] [pid 1] AH00163: Apache/2.4.25 (Debian) PHP/7.3.15 configured -- resuming normal operations
drupal_1    | [Sat May 23 21:00:35.997964 2020] [core:notice] [pid 1] AH00094: Command line: 'apache2 -D FOREGROUND'
postgres_1  | running bootstrap script ... ok
postgres_1  | performing post-bootstrap initialization ... ok
postgres_1  | syncing data to disk ... ok
postgres_1  |
postgres_1  |
postgres_1  | Success. You can now start the database server using:
postgres_1  |
postgres_1  |     pg_ctl -D /var/lib/postgresql/data -l logfile start
postgres_1  |
postgres_1  | initdb: warning: enabling "trust" authentication for local connections
postgres_1  | You can change this by editing pg_hba.conf or using the option -A, or
postgres_1  | --auth-local and --auth-host, the next time you run initdb.
postgres_1  | waiting for server to start....2020-05-23 21:00:37.032 UTC [45] LOG:  starting PostgreSQL 12.1 (Debian 12.1-1.pgdg100+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 8.3.0-6) 8.3.0, 64-bit
postgres_1  | 2020-05-23 21:00:37.035 UTC [45] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
postgres_1  | 2020-05-23 21:00:37.052 UTC [46] LOG:  database system was shut down at 2020-05-23 21:00:36 UTC
postgres_1  | 2020-05-23 21:00:37.060 UTC [45] LOG:  database system is ready to accept connections
postgres_1  |  done
postgres_1  | server started
postgres_1  |
postgres_1  | /usr/local/bin/docker-entrypoint.sh: ignoring /docker-entrypoint-initdb.d/*
postgres_1  |
postgres_1  | 2020-05-23 21:00:37.122 UTC [45] LOG:  received fast shutdown request
postgres_1  | waiting for server to shut down....2020-05-23 21:00:37.125 UTC [45] LOG:  aborting any active transactions
postgres_1  | 2020-05-23 21:00:37.130 UTC [45] LOG:  background worker "logical replication launcher" (PID 52) exited with exit code 1
postgres_1  | 2020-05-23 21:00:37.131 UTC [47] LOG:  shutting down
postgres_1  | 2020-05-23 21:00:37.155 UTC [45] LOG:  database system is shut down
postgres_1  |  done
postgres_1  | server stopped
postgres_1  |
postgres_1  | PostgreSQL init process complete; ready for start up.
postgres_1  |
postgres_1  | 2020-05-23 21:00:37.248 UTC [1] LOG:  starting PostgreSQL 12.1 (Debian 12.1-1.pgdg100+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 8.3.0-6) 8.3.0, 64-bit
postgres_1  | 2020-05-23 21:00:37.248 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
postgres_1  | 2020-05-23 21:00:37.249 UTC [1] LOG:  listening on IPv6 address "::", port 5432
postgres_1  | 2020-05-23 21:00:37.253 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
postgres_1  | 2020-05-23 21:00:37.276 UTC [54] LOG:  database system was shut down at 2020-05-23 21:00:37 UTC
postgres_1  | 2020-05-23 21:00:37.282 UTC [1] LOG:  database system is ready to accept connections
drupal_1    | 172.20.0.1 - - [23/May/2020:21:00:54 +0000] "GET / HTTP/1.1" 302 609 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36"
drupal_1    | 172.20.0.1 - - [23/May/2020:21:00:54 +0000] "GET /core/install.php HTTP/1.1" 200 4279 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36"
drupal_1    | 172.20.0.1 - - [23/May/2020:21:00:55 +0000] "GET /core/assets/vendor/normalize-css/normalize.css?0 HTTP/1.1" 200 2916 "http://localhost:8080/core/install.php" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36"
drupal_1    | 172.20.0.1 - - [23/May/2020:21:00:55 +0000] "GET /core/misc/normalize-fixes.css?0 HTTP/1.1" 200 534 "http://localhost:8080/core/install.php" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36"
```
# Adding Image Building to Compose Files
![using_compose_to_build](https://github.com/NoriKaneshige/Docker_Compose_Multi_Container_Tool/blob/master/using_compose_to_build.png)
## docker-compose.yml
### When I run the docker compose up command, it's going to first check if there is a specified image in my cache. If it doesn't find it, then it's going to use these build commands (build: in proxy) to look up the dockerfile and build my image. Once it is built the first time, it will not need to be really rebuilt very often. When I'm editing and developing on my website live, I have the configuration in the volumes in the web container part.
```
version: '2'

# based off compose-sample-2, only we build nginx.conf into image
# uses sample site from https://startbootstrap.com/template-overviews/agency/

services:
  # two services below
  # nginx proxy
  proxy:
    # here, instead of specifying the default image for nginx, we can build a custom one
    build:
      # I want it to build that dockerfile in this current directory.
      context: .
      # I'm telling the dockerfile I need to use
      dockerfile: nginx.Dockerfile
      # we can name image the image when it is built, but if we want to use docker-compose down -irm to delete
      # the image automatically, we we don't need it here
      # image: nginx-custom
    ports:
      - '80:80'

  # apache web server
  web:
    image: httpd
    volumes:
      # we actually mount html source files into apache server
      - ./html:/usr/local/apache2/htdocs/
```
## nginx.Dockerfile
```
FROM nginx:1.13

COPY nginx.conf /etc/nginx/conf.d/default.conf
```
## Let's do docker-compse up
![using_compose_to_build_2](https://github.com/NoriKaneshige/Docker_Compose_Multi_Container_Tool/blob/master/using_compose_to_build_2.png)
```
Koitaro@MacBook-Pro-3 compose-sample-3 % docker-compose up
Creating network "compose-sample-3_default" with the default driver
Building proxy
Step 1/2 : FROM nginx:1.13
 ---> ae513a47849c
Step 2/2 : COPY nginx.conf /etc/nginx/conf.d/default.conf
 ---> d73f50db5cf6
Successfully built d73f50db5cf6
Successfully tagged compose-sample-3_proxy:latest
WARNING: Image for service proxy was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating compose-sample-3_proxy_1 ... done
Creating compose-sample-3_web_1   ... done
Attaching to compose-sample-3_web_1, compose-sample-3_proxy_1
web_1    | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.22.0.2. Set the 'ServerName' directive globally to suppress this message
web_1    | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.22.0.2. Set the 'ServerName' directive globally to suppress this message
web_1    | [Sat May 23 21:28:24.618488 2020] [mpm_event:notice] [pid 1:tid 140021325591680] AH00489: Apache/2.4.43 (Unix) configured -- resuming normal operations
web_1    | [Sat May 23 21:28:24.619824 2020] [core:notice] [pid 1:tid 140021325591680] AH00094: Command line: 'httpd -D FOREGROUND'
proxy_1  | 172.22.0.1 - - [23/May/2020:21:28:40 +0000] "GET / HTTP/1.1" 200 35099 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36" "-"
web_1    | 172.22.0.3 - - [23/May/2020:21:28:40 +0000] "GET / HTTP/1.0" 200 35099
web_1    | 172.22.0.3 - - [23/May/2020:21:28:40 +0000] "GET /css/agency.min.css HTTP/1.0" 200 13277
proxy_1  | 172.22.0.1 - - [23/May/2020:21:28:40 +0000] "GET /css/agency.min.css HTTP/1.1" 200 13277 "http://localhost/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36" "-"
proxy_1  | 172.22.0.1 - - [23/May/2020:21:28:40 +0000] "GET /lib/font-awesome/css/font-awesome.min.css HTTP/1.1" 200 31000 "http://localhost/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36" "-"
web_1    | 172.22.0.3 - - [23/May/2020:21:28:40 +0000] "GET /lib/font-awesome/css/font-awesome.min.css HTTP/1.0" 200 31000
web_1    | 172.22.0.3 - - [23/May/2020:21:28:40 +0000] "GET /lib/tether/tether.min.js HTTP/1.0" 200 24989
proxy_1  | 172.22.0.1 - - [23/May/2020:21:28:40 +0000] "GET /lib/tether/tether.min.js HTTP/1.1" 200 24989 "http://localhost/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36" "-"
web_1    | 172.22.0.3 - - [23/May/2020:21:28:40 +0000] "GET /lib/bootstrap/css/bootstrap.min.css HTTP/1.0" 200 150996
proxy_1  | 172.22.0.1 - - [23/May/2020:21:28:40 +0000] "GET /lib/bootstrap/css/bootstrap.min.css HTTP/1.1" 200 150996 "http://localhost/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36" "-"
proxy_1  | 172.22.0.1 - - [23/May/2020:21:28:40 +0000] "GET /lib/bootstrap/js/bootstrap.min.js HTTP/1.1" 200 46653 "http://localhost/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.138 Safari/537.36" "-"
```
## Now, because I bind mounted that directory, I can edit the files live
![edited_live](https://github.com/NoriKaneshige/Docker_Compose_Multi_Container_Tool/blob/master/edited_live.png)
## Let's stop and clean up
## Note that it actually names containers, volumes, networks with the name of the directory as the project name to prevent name conflicts. It will always add the directory name on the beginning of all of the assets. Also we can use docker-compose down --rmi command to remove network, images etc.
```
Gracefully stopping... (press Ctrl+C again to force)
Stopping compose-sample-3_proxy_1 ... done
Stopping compose-sample-3_web_1   ... done

Koitaro@MacBook-Pro-3 compose-sample-3 % docker-compose down
Removing compose-sample-3_proxy_1 ... done
Removing compose-sample-3_web_1   ... done
Removing network compose-sample-3_default

# actually it doesn't delete image by default
Koitaro@MacBook-Pro-3 compose-sample-3 % docker image ls
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
compose-sample-3_proxy    latest              d73f50db5cf6        31 minutes ago      109MB

Koitaro@MacBook-Pro-3 compose-sample-3 % docker-compose down --help
Stops containers and removes containers, networks, volumes, and images
created by `up`.

By default, the only things removed are:

- Containers for services defined in the Compose file
- Networks defined in the `networks` section of the Compose file
- The default network, if one is used

Networks and volumes defined as `external` are never removed.

Usage: down [options]

Options:
    --rmi type              Remove images. Type must be one of:
                              'all': Remove all images used by any service.
                              'local': Remove only images that don't have a
                              custom tag set by the `image` field.
    -v, --volumes           Remove named volumes declared in the `volumes`
                            section of the Compose file and anonymous volumes
                            attached to containers.
    --remove-orphans        Remove containers for services not defined in the
                            Compose file
    -t, --timeout TIMEOUT   Specify a shutdown timeout in seconds.
                            (default: 10)

Koitaro@MacBook-Pro-3 compose-sample-3 % docker-compose up -d
Creating network "compose-sample-3_default" with the default driver
Creating compose-sample-3_proxy_1 ... done
Creating compose-sample-3_web_1   ... done

Koitaro@MacBook-Pro-3 compose-sample-3 % docker-compose down --rmi local
Stopping compose-sample-3_web_1   ... done
Stopping compose-sample-3_proxy_1 ... done
Removing compose-sample-3_web_1   ... done
Removing compose-sample-3_proxy_1 ... done
Removing network compose-sample-3_default
Removing image compose-sample-3_proxy

Koitaro@MacBook-Pro-3 compose-sample-3 % docker image ls
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
```
