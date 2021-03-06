
**Some commands that we will use in this workshop**
=====================================================




**Managing containers:**
--------------------

Create our First Container:
--------------------

`docker create -t -i --name firstContainer debian`

**Start Container:**

`docker start  firstContainer`


**We can create a container and run directly by:**

`docker run -ti  --name SecondContainer --cpu-shares 1000 --memory 512m centos`

**Our first Container we forgot to add cpu-shares and  memory,**

**Let's modify it now:**

`docker update  firstContainer --cpu-shares 1000 --memory 512 `

**Let's check the updates:**

`docker inspect firstContainer | grep -i cpu`


`docker inspect firstContainer | grep -i memory`

**CTRL + P + Q: to leave the container  without kill the container:**
**CTRL + Q : To Kill the container**

**We can see all the containers running by:**

`docker ps`

**See all containers** 

`docker ps -a`

**And to return to the container:**

`docker attach {CONTAINER ID or NAME}`

**To Stop,Start,Pause,Unpause a container we have to do:**

`docker start {Container ID or Name}` **just replace start for what you want to do**


## Volumes and DATA-ONLY:
------------------

**Our first  volumes :**

`docker run -ti -v /volume ubuntu /bin/bash`

**We can see if our volume is mounted by running:**

`df -h`

**Obviously, we can see  by accessing :**

`cd /`

`ls -lha`

**Create a  file or a folder in volumes:**

`cd volume  && touch DockerWorkshop`

**Now you can leave the container without kill the container by:**

### CRTL + P  + Q

**Now to find where our volume is mounted you can do:**

`docker inspect  {CONTAINER ID or Name} -f {{.Mounts}}`

**Now enter inside the volume and you can see your file or folder**

`cd /var/lib/docker/volumes/b127e735f50f0eb2ccf930593142e2d05c5781f93d81d0993caaaf39ac2784e5/_data`

**Now we will to create a new volume but i don't whant  that he mounts  wherever we wants

 **In my machine :**

`mkdir /root/First_Dockerfile`

**Run:**

`docker run -ti -v /root/First_Dockerfile:volume ubuntu`

`cd /volume`

**Create a Dockerfile we use it later.**

`touch Dockerfile`

### CTRL + P + Q

 **If we now access `cd /root/First_Dockerfile` we can see our Dockerfile.**


## **Data-only**
-------------------------------

**We gona create or db data to share  with others containers:**

`docker create -v/data --name dbdata centos`


**Let's create two containeres to use the same  data:**


`docker run -d -p 5432:5432 --name pgsql1 --volumes-from dbdados -e POSTGRESQL_USER=docker -e POSTGRESQL_PASS=docker kamui/postgresql`

`docker run -d -p 5432:5433 --name pgsql2 --volumes-from dbdados -e POSTGRESQL_USER=docker -e POSTGRESQL_PASS=docker kamui/postgresql`


**run** `docker inspect -f {{.Mounts}} dbdados`


## **Dockerfile:**
-------------------------

**First Dockerfile:**
 `cd /root/First_Dockerfile`
 `touc index.html`
 `vim Dockerfile`
 
 <-- FROM debian

MAINTAINER Diogo Martins

RUN apt update && apt install apache && apt clean

COPY index.html  /var/www/html/

ENTRYPOINT ["/usr/bin/apache2ctl", "-D", "FOREGROUND"]

EXPOSE 80 -->

**Save the dockerfile and run:**


`docker build .`

`docker images`

`docker build -t firsImage:1.0 .`



## **Docker hub:**
---------------------------------------
Let's create our account in Docker Hub:
https://hub.docker.com/
 **Before using a container a run this command first**

`docker inspect {NAME}`

 **And:**

`docker history {NAME}`

**Let's push our first image:**

 **First we need to tag our image:**

`docker tag {CONTAINER ID or Name} {USERNAME of DOCKERHUB}/{NAME TO OUR IMAGE}:1.0`
**Let's make sure our image is created:**

`docker images | grep {NAMEofOURImage}`

**Login to Docker Hub:**

`docker login`

 **Let's push our  Image:**

`docker push {NameofUser}/{NameOFImage}:1.0` {1.0 === version}

 **Let's search all images we have by using this command:**

`docker search {UserName of Docker Hub}`

 **And now to pull one image :**

`docker pull {UserName of DockerHub}/{CONTAINER ID OR NAME AND VERSION}` =>  diogomamartins/webserver:1.0


## **Portainer**/
------------------------------------


 **Portainer is a UI to docker to help us :**

`docker  run -d -p  9000:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer`

### Go to localhost:9000

## **Local Registry:**
----------------------------
`docker run -d -p 5000:5000 --restart=always --name registry registry:2`


`docker tag aec1dc74cf14 localhost:5000/first_image:1.0`

**Push our image to our registry:**

`docker push  localhost:5000/first_image:1.0`


**To see all the images we have in registry :**

`curl localhost:5000/v2/_catalog`


 **Configuring network containers**

`ifconfig `

**We usually have docker0 that is our brigde**

`docker run -ti --dns 8.8.8.8 debian`

`cat /etc/resolv.conf`


**Let's create a container:**

`docker run -ti --name container1 debian`

**Let's create a connection between container1 and container2 and create container2 :**

`docker run -ti --link container1 --name container2 debian`

**In container two you can send a ping to container1:**


`ping container1`

`cat /etc/hosts`

**Basic commands:**
    
`docker run -ti --expose 80`

`docker run -ti --publish 8080:80 debian`

`docker run -ti --mac-address 12:34:de:b0:6b:61 debian`

`ip addr`

**Let's make a redirect to apache2**

`docker run -ti -p 8080:80 --name  apacheSwartz debian`

`apt install apache2`

**Let's start apache**

`/etc/init.d/apache2 start`

`iptables -t nat -L -n`

**If we don't want to use docker0 ,we can use the host by :**

`docker run -ti --net=host debian`


**Docker machine**
-----------------------

**For use Docker machine we need to install virtual box and docker-machine:**

[https://docs.docker.com/machine/install-machine/](URL)

[https://www.virtualbox.org/wiki/Downloads](URL)

**First Driver:**
`docker-machine create --driver virtualbox swartz`

`docker-machine env swartz`

`eval $(docker-machine env swartz)`

`docker ps`

**Show all the host available:**

`docker-machine ls`


`docker run busybox echo "swartz "`


`docker.machine ip swartz `

`docker-machine ssh swartz`

`docker-machine stop swartz`

`docker-machine ls`

`docker-machine rm swartz`

**Docker Compose**

We need to install docker-compose in ubuntu:

`apt install docker-compose`

`https://docs.docker.com/compose/install/`


Let's create a folder:

`mkdir compose && cd compose`

Create a file docker-compose:

`vim docker-compose.yml`

 db:
  image: postgres
web:
  build:  .
  command: python manage.py runserver 0.0.0:8000
  volumes:
    - .:/code
  ports:
    - 8000:8000
  links:
    - db

Create now a Dockerfile:

`vim Dockerfile`:

FROM python:2.7
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
ADD requirements.txt /code/
RUN pip install -r requirements.txt
ADD . /code/

Now we have to create requirements file:

`vim requirements.txt`

Now we can RUN:


`docker-compose run web django-admin.py startproject example .`

Enter inside the folder example:

`vim settings.py`

and add:

    DATABASES = {
    'default':{
    .......
    'USER':'postgres',
    'HOST':'db',
    'PORT':5432,
    }
}

Save and return to the  folder an run docker-compose

`cd ..`

`docker-compose up`

What our django:

`curl localhost:8000`

If we want to scale our application:

`docker-compose scale db=2`


`docker-compose stop`






















