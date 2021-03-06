## Table of Contents
**[Docker](#docker)**<br>
**[Dockerfile](#dockerfile)**<br>
**[Compose](#compose)**<br>
**[Development and Production instances](#dev-and-prod-instances)**<br>

## Docker
* Runs an image from the local repository and exits immediately.

```shell
$ sudo docker run localhost:5000/local-centos
$ 
```

* Runs an image from the local repostory and gets an interactive TTY shell.

```shell
$ sudo docker run -i -t localhost:5000/local-centos 
[root@6bbf8236255a /]# 
```

* Run an image from the local repository. Start with __-d__ and __-it__ to run in daemon mode and be able to reattach.

```shell
$ s docker run -d -it localhost:5000/local-centos bash
000c8a767f2754ac1ad6cbf62c85922b79cf85a2ea163eda888d45a78d568581

$ s docker ps
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS              PORTS                    NAMES
000c8a767f27        localhost:5000/local-centos   "bash"                   4 seconds ago       Up 3 seconds                                 dazzling_hodgkin
d51ca36e5f42        registry:2                    "/entrypoint.sh /etc…"   37 hours ago        Up 37 hours         0.0.0.0:5000->5000/tcp   local-repository
```

* Show the ports exposed by a specific container. Note: `docker ps` or `docker container ls` lists the ports in order of external then internal port, whereas `docker port` lists the ports in order of internal then external ports.

```shell
$ docker port nginx
```

* Reattach to the instance running in the background. _Exits_ and stops the running instance.

```shell
$ s docker attach 00
[root@000c8a767f27 /]# exit
```

* Reattach to the instance running in the background. __Ctrl-p__ + __Ctrl-q__ detaches from instance and leaves it running in the background.

```shell
$ s docker attach 00
[root@000c8a767f27 /]# read escape sequence 
```

* Find the IP address of a running container.

```shell
$ s docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
d72a1dd7e14e        debian              "sleep 3600"        5 minutes ago       Up 5 minutes                            trusting_hoover

$ s docker inspect d7 | grep "IPAddress"
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
                    
$ ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.081 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.100 ms
^C
--- 172.17.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1009ms
rtt min/avg/max/mdev = 0.081/0.090/0.100/0.013 ms

$ s docker stop d7
d7

$ s docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

$ s docker inspect d7 | grep "IPAddress"
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "",
```

## Commmit, Save and Load Images.

* Pull a _reference_ image from hub.docker.com.

```shell
$ docker pull httpd
Using default tag: latest
latest: Pulling from library/httpd
f189db1b88b3: Pull complete 
ba2d31d4e2e7: Pull complete 
23a65f5e3746: Pull complete 
5e8eccbd4bc6: Pull complete 
4c145eec18d8: Pull complete 
1c74ffd6a8a2: Pull complete 
1421f0320e1b: Pull complete 
Digest: sha256:8631904c6e92918b6c7dd82b72512714e7fbc3f1a1ace2de17cb2746c401b8fb
Status: Downloaded newer image for httpd:latest
```

```shell
$ docker images -a | { head -1; grep [h]ttpd; }
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
httpd               latest              d595a4011ae3        5 days ago          178 MB
```

* Run the local image, then make a change to the running container.

```shell
$ docker run -it --name httpd-template-base -p 8080:80 -e TERM=xterm -d httpd
86a43e482df77319fed13db5ff3f600ca2293c0d2b20fb1b1d41c2d30586aa63

OR

$ docker run -it --env HTTP_PROXY="http://proxy.corporate.com:81" --name httpd-template-base2 -p 8080:80 -e TERM=xterm -d httpd
2c5fb262a533813ff6026a9ba1bc64622cba872f9b96326bd38fa25eafc90815


$ docker ps -a | { head -1; grep [h]ttpd; }
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS                  NAMES
86a43e482df7        httpd               "httpd-foreground"       23 seconds ago      Up 23 seconds              0.0.0.0:8080->80/tcp   httpd-template-base
```

* Connect interactively to the container running in the background and make changes to it.

```shell
$ docker exec -it 86 bash
root@2c5fb262a533:/usr/local/apache2# export http_proxy="http://proxy.corporate.com:81"
root@2c5fb262a533:/usr/local/apache2# apt-get update                              
Get:1 http://security.debian.org jessie/updates InRelease [44.9 kB]
Ign http://deb.debian.org jessie InRelease                 
Get:2 http://deb.debian.org jessie-updates InRelease [145 kB]
Get:3 http://security.debian.org jessie/updates/main amd64 Packages [656 kB]
Get:4 http://deb.debian.org jessie-backports InRelease [166 kB]                   
Ign http://deb.debian.org stretch InRelease
Get:5 http://deb.debian.org jessie Release.gpg [2420 B]
Get:6 http://deb.debian.org jessie-updates/main amd64 Packages [23.0 kB]
Get:7 http://deb.debian.org stretch Release.gpg [2434 B]
Get:8 http://deb.debian.org jessie Release [148 kB]
Get:9 http://deb.debian.org jessie-backports/main amd64 Packages [1172 kB]
Get:10 http://deb.debian.org stretch Release [118 kB]
Get:11 http://deb.debian.org jessie/main amd64 Packages [9098 kB]
Get:12 http://deb.debian.org stretch/main amd64 Packages [9500 kB]
Fetched 21.1 MB in 1min 4s (325 kB/s)
Reading package lists... Done
root@2c5fb262a533:/usr/local/apache2# apt-get install vim
...
```

* Exit the container and commit the changes to a new image.

```shell
root@2c5fb262a533:/usr/local/apache2# exit
$ 

$ docker commit 2c httpd-template
sha256:69b333de9627c43e4005cf33398840b65ddb362191e24d8d89f6ad10cc9ae8bb
$ 

$ docker images | { head -1; grep [h]ttpd; }
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
httpd-template      latest              69b333de9627        39 seconds ago      227 MB
$
```

* Save the new image as a `.tar` archive. The `.tar` archive can be scp'd to another host and brought up as a container.
```shell
$ docker save -o http-template.tar httpd-template

$ ls *.tar
http-template.tar
```

* Use `docker load` to import the `.tar` archive as an image.

```shell
$ docker load -i http-template.tar 
Loaded image: httpd-template:latest
```

### Cleaning Up

* Cleaning up stopped containers.

```shell
docker rm -v $(docker ps -aq -f status=exited)
```

* Kill all running containers.
```shell
docker kill $(docker ps -q)
```

* Delete all stopped containers.

```shell
docker rm $(docker ps -a -q)
```

* Delete all images.

```shell
docker rmi $(docker images -q)
```

* Prune all _dangling_ images.

```shell
docker rmi $(docker images -f "dangling=true" -q)
```

## Dockerfile

* **FROM** instruction specifies the base image to use. All `Dockerfiles` must have a **FROM** instruction as the first non-comment instruction. 
* **RUN** instructions specify a shell command to execute inside the image.
```shell
$ cat Dockerfile 
FROM ubuntu

RUN apt-get update && apt-get install -y python python-pip

RUN pip -v install --trusted-host=pypi.org --trusted-host=files.pythonhosted.org --proxy=http://proxy.domain.com:81 flask

COPY app.py /opt/

ENTRYPOINT FLASK_APP=/opt/app.py flask run --host=0.0.0.0 --port=80
```
* *Build* the image by running `docker build` inside the same directory where the `Dockerfile` lives. Use `--no-cache` to do a clean build without relying on the cache from the last build.
```shell
docker build --no-cache --build-arg http_proxy=http://proxy.domain.com:81 -t my-simple-webapp .
```
* *Run* the image by typing `docker run -it <image_name:tag> [command_to_be_executed_on_image]`.

```shell
$ docker run -it mydjango:v0.1
root@75d08980aade:/# 
root@75d08980aade:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@75d08980aade:/# exit
```
* Start the image as a *daemon* and connect to the running container.

```shell
$ docker run -it -d mydjango:v0.1
574da457a562e129027c07fb0acfc21eab55e9d0529f0b2a856525586bd45e37
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS           PORTS     NAMES
574da457a562        mydjango:v0.1       "/bin/bash"         13 seconds ago      Up 13 seconds              hopeful_hypatia
$ docker attach 574da457a562
root@574da457a562:/# 
```

* Execute commands in a running container and exit.

```shell
$ docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                    NAMES
7c971d9f0929        55e83ce465d6                "/bin/sh -c '/bin/..."   6 minutes ago       Up 6 minutes                                 relaxed_sammet
```

```shell
$ docker exec  7c971d9f0929 ls -la /repo/
total 20
dr-xrwx--x  4 root www-data 4096 Aug 17 05:04 .
drwxr-xr-x 51 root root     4096 Aug 17 05:02 ..
drwxr-xr-x  4 root root     4096 Aug 17 05:04 .temp
-r--r--r--  1 root root       15 Aug 17 05:02 Archive-Update-in-Progress-7c971d9f0929
drwxr-xr-x  5 root root     4096 Aug 17 05:05 dists
-rw-r--r--  1 root root        0 Aug 17 04:55 test.txt
```

## Compose

* `docker-compose` is a wrapper around the `docker` command. `docker-compose build` will read the `docker-compose.yml` file, look for all `services` containing the `build:` statement and run a `docker build` for each one.
* Only builds the images, does not start the containers:

```shell
docker-compose build
```

* Builds the images _if the images do not exist_ and starts the containers for the `services` in the `docker-compose.yml` file. If you make changes to the `Dockerfile` that the image is based on and want the image to inherit the changes, you can just add the `--build` option instead of manually deleting the image and rerunning.

```shell
docker-compose up
```

* Forces a build of the images even when it is not needed (like using the cache). _Ordering_ of the flags are significant.

```shell
docker-compose --verbose up --build
```

* Running the _development server_.

```shell
$ docker-compose up --build
Building hello
Step 1/8 : FROM python:3
 ---> 825141134528
Step 2/8 : ENV PYTHONUNBUFFERED 1
 ---> Using cache
 ---> b90b94a5958d
Step 3/8 : WORKDIR /app
 ---> Using cache
 ---> 897498d3ff70
Step 4/8 : COPY app.py /app
 ---> Using cache
 ---> dba72def05e5
Step 5/8 : COPY requirements.txt /app
 ---> Using cache
 ---> 2a830a50f513
Step 6/8 : RUN pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org --proxy http://proxy.bloomberg.com:81 -r requirements.txt
 ---> Using cache
 ---> 26a49354bbbe
Step 7/8 : EXPOSE 80
 ---> Using cache
 ---> 326c8e1fb02c
Step 8/8 : CMD python app.py
 ---> Using cache
 ---> 5352b7d87725
Successfully built 5352b7d87725
Recreating flaskapp_hello_1
Attaching to flaskapp_hello_1
hello_1  |  * Serving Flask app "app" (lazy loading)
hello_1  |  * Environment: production
hello_1  |    WARNING: Do not use the development server in a production environment.
hello_1  |    Use a production WSGI server instead.
hello_1  |  * Debug mode: on
hello_1  |  * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
hello_1  |  * Restarting with stat
hello_1  |  * Debugger is active!
hello_1  |  * Debugger PIN: 197-442-924
hello_1  | 10.144.24.78 - - [04/Aug/2018 15:34:37] "GET /how%20are%20you? HTTP/1.1" 200 -
```

* Starts the containers in _detached_ mode and run them in the background. 

```shell
docker-compose up -d
```

* Check if the containers are running.

```shell
$ docker-compose ps
     Name           Command      State         Ports        
-----------------------------------------------------------
flaskapp_web_1   python app.py   Up      0.0.0.0:80->80/tcp 
```

* Check the console logs of the detached containers.

```shell
docker-compose logs
```

* Stop the running containers.

```shell
$ docker-compose ps        
     Name           Command      State         Ports        
-----------------------------------------------------------
flaskapp_web_1   python app.py   Up      0.0.0.0:80->80/tcp 
$ docker-compose stop
Stopping flaskapp_web_1 ... done
$ docker-compose ps  
     Name           Command      State    Ports 
-----------------------------------------------
flaskapp_web_1   python app.py   Exit 0
```

* Delete the stopped containers.

```shell
$ docker-compose ps  
     Name           Command      State    Ports 
-----------------------------------------------
flaskapp_web_1   python app.py   Exit 0         
$ docker-compose down
Removing flaskapp_web_1 ... done
Removing network flaskapp_default
$ docker-compose ps  
Name   Command   State   Ports 
------------------------------
```

* To ensure we can `attach` to a container already running in the background, ensure `stdin_open: true` and `tty: true` are configured in `docker-compose.yml`.

```shell
        stdin_open: true
        tty: true
```

* Run a container in the background with the `-d` flag. If `stdin_open: true` and `tty: true` are configured in `docker-compose.yml` you can attach to the container and _detaach_ from it with `Ctrl-p` + `Ctrl-q` without killing the container.

```shell
$ docker-compose up --build -d
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
9f24ee3ba251        nginxdocker_nginx   "nginx -g 'daemon ..."   33 seconds ago      Up 32 seconds       0.0.0.0:80->80/tcp   nginx_prod
$ docker attach nginx_prod
10.144.24.78 - - [16/Aug/2018:02:49:46 +0000] "GET /guacamole/ HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36" "-"
10.144.24.78 - - [16/Aug/2018:02:49:46 +0000] "GET /guacamole/api/languages?token=85875af0ebba63c00dac313322a9328e38fb98eb2573bdc2c31a3da9f06cfba7 HTTP/1.1" 200 115 "http://10.150.139.249/guacamole/" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36" "-"
10.144.24.78 - - [16/Aug/2018:02:49:46 +0000] "POST /guacamole/api/tokens HTTP/1.1" 403 161 "http://10.150.139.249/guacamole/" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36" "-"
10.144.24.78 - - [16/Aug/2018:02:49:46 +0000] "GET /guacamole/images/logo-144.png HTTP/1.1" 200 9167 "http://10.150.139.249/guacamole/" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36" "-"
```

* Use `volumes` in `docker-compose.yml` to mount files on the host to a location in a container. Any changes to the file on the host will only be reflected in the container after running `docker restart <container_id>`. There will be a disconnection.

```shell
        volumes:
            - ./app/user-mapping.xml:/etc/guacamole/user-mapping.xml
```

## Development and Production instances

### Development

* Running a development environment on high-ports and with these files modified to a certain extend.

```shell
.
+-- apps
|   +-- app.py <- Modify for development.
|   +-- Dockerfile <- Modify for development.
|   +-- requirements.txt
|   +-- templates
|   |   +-- index.html
|   +-- uwsgi.ini <- Not required for development.
+-- docker-compose.yml <- Modify for development.
+-- nginx
    +-- Dockerfile <- Not required for development.
    +-- nginx.conf <- Not required for development.
```

```shell
$ grep 5000 apps/app.py    
    app.run(host="0.0.0.0", port='5000', debug=True)
```

```shell
$ egrep "^CMD" apps/Dockerfile              
CMD ["python", "app.py"]
```

```shell
$ cat docker-compose.yml 
version: '2'
services:
    webui:
        build: ./apps
        ports:
            - "5000:5000"

    #nginx:
    #    build: ./nginx
    #    volumes:
    #        - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    #    ports:
    #        - "81:81"
    #    links:
    #        - webui
```

* Go into the project directory and run `docker-compose ps` to see the running containers. In the dev directory you'll be able to see it's running containers only.

```shell
$ docker-compose ps
       Name              Command      State           Ports          
--------------------------------------------------------------------
flaskappdev_webui_1   python app.py   Up      0.0.0.0:5000->5000/tcp 
```

* `docker ps` will show the global pool of running containers.

```shell
$ docker ps
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                    NAMES
2299a9e0015b        flaskappdev_webui    "python app.py"          4 minutes ago       Up 4 minutes        0.0.0.0:5000->5000/tcp   flaskappdev_webui_1
de447f5a75df        flaskappprod_nginx   "nginx -g 'daemon ..."   16 hours ago        Up 16 hours         0.0.0.0:80->80/tcp       flaskappprod_nginx_1
53f46ed54816        flaskappprod_webui   "uwsgi --ini /app/..."   16 hours ago        Up 16 hours         0.0.0.0:3031->3031/tcp   flaskappprod_webui_1
```

* Testing application in development environment by temporarily resetting `http_proxy` environment variable.

```shell
$ http_proxy='' curl -vvv -L -k curl -H "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36" http://127.0.0.1:5000/hello/foo
```

* Limit container resources in the `docker-compose.yml` file.

```shell
$ cat docker-compose.yml 
version: '2'
services:
    webui:
        build: ./apps
        container_name: "flaskapp_prod_webui"
        ports:
            - "3031:3031"

    nginx:
        build: ./nginx
        container_name: "flaskapp_prod_nginx"
        volumes:
            - ./nginx/nginx.conf:/etc/nginx/nginx.conf
        ports:
            - "80:80"
        links:
            - webui
        mem_limit: 100000000 <- limits to 100MB!
```

```shell
$ docker stats fa2b6dedadd9
CONTAINER     CPU %     MEM USAGE / LIMIT      MEM %    NET I/O           BLOCK I/O   PIDS
c46d96b4e41a  0.00%     1.43 MiB / 190.7 MiB   0.75%    1.74 kB / 648 B   0 B / 0 B   2
                                   ^^^^^^^^^
```

### Production

* Docker production directory layout and configuration files.

```shell
.
+-- apps
|   +-- app.py
|   +-- Dockerfile
|   +-- requirements.txt
|   +-- templates
|   |   +-- index.html
|   +-- uwsgi.ini
+-- docker-compose.yml
+-- nginx
    +-- Dockerfile
    +-- nginx.conf
```

* Bring up the containers in the background (daemon mode) with the `-d` flag.

```shell
docker-compose up --build -d
```

* List all containers that are running, either in the foreground or background.

```shell
$ docker-compose ps
       Name           Command                          State   Ports          
-------------------------------------------------------------------------------------
flaskapp_prod_nginx   nginx -g daemon off; -c /e ...   Up      0.0.0.0:80->80/tcp     
flaskapp_prod_webui   uwsgi --ini /app/uwsgi.ini       Up      0.0.0.0:3031->3031/tcp 
```

* In order to _execute_ commands on the containers running in the background, use one of these `docker` commands.

```shell
$ docker ps
CONTAINER ID      IMAGE           COMMAND                  CREATED             STATUS          PORTS                    NAMES
0d7959f678d5      prod_nginx      "nginx -g 'daemon ..."   42 seconds ago      Up 41 seconds   0.0.0.0:80->80/tcp       flaskapp_prod_nginx
cfffe567b1d9      prod_webui      "uwsgi --ini /app/..."   42 seconds ago      Up 41 seconds   0.0.0.0:3031->3031/tcp   flaskapp_prod_webui
```

```shell
$ docker exec -it cfffe567b1d9 ls -lFR /app
/app:
total 20
-rw-rw-r-- 1 root root  399 Aug  8 12:39 app.py
-rw-rw-r-- 1 root root   27 Aug  6 08:51 requirements.txt
drwxr-xr-x 2 root root 4096 Aug  9 06:13 static/
drwxr-xr-x 2 root root 4096 Aug  9 06:13 templates/
-rw-rw-r-- 1 root root  197 Aug  9 06:02 uwsgi.ini

/app/static:
total 4
-rw-rw-r-- 1 root root 28 Aug  8 14:23 style.css

/app/templates:
total 12
-rw-rw-r-- 1 root root 488 Aug  9 02:02 base.html
-rw-rw-r-- 1 root root 428 Aug  9 01:43 index.html
-rw-rw-r-- 1 root root 335 Aug  9 02:04 welcome.html
```

* To _attach_ to a container running in the background do the following. Notice that the restricted shell is a good security measure.

```shell
$ docker exec -it cfffe567b1d9 bash
root@cfffe567b1d9:/app# ifconfig
bash: ifconfig: command not found
root@cfffe567b1d9:/app# reboot
bash: reboot: command not found
```
