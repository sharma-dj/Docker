# Docker Notes

# 1) What is a docker container?
Docker container is a way to package the application with all the dependencies and configurations.
Technically, Docker container is group of layers.

# 2) Architecture of OS?
Os consists of 3 layers. Hardware, OS kernal and Application layer. Os kernal and Application are called layer1 and layer2 respectively. 
Os kernal is the one that comunicate with the hardware. Applications run on the top of OS kernal.

# 3) Difference between Docker and VM.
Docker run on only application layer of the host computer. It uses the OS kernal of the host computer. That's why it is fast and takes smaller space. 
VM has its own application layer and OS kernal layer. It only uses the hardwares of the host computer. That's why it is slow and require larger space. 
On every time VM is started it bootup its own kernal. Since Docker uses the OS kernal of the host, so Docker software for Windows, Mac and Linux are different.

# 4) Difference between DockerImage and DockerContainer.
DockerImage : DockerImage is the actual package that consists of docker image, its configuration and starting script. When we pull some docker image on our 
computer and it is not yet started, then it just a DockerImage.
DockerContainer: When a DockerImage has been started on the machine it is called as DockerContainer. So, a DockerImage in running state is DockerContainer.

# 5) List of docker images command.
docker images

The above command shows the list of all the docker images on our machine irrespective of whether they were started anytime of not. So, we can remove those of unwanted images if we like.

# 6) List of docker containers command.

docker ps : This command lists all of the running containers.
docker ps -a : This command lists all of the containers whether they are running or not.

# 7) How to pull a docker image from docker hub?
docker pull <image_name> => It will pull the latest version of the image
docker pull <image_name>:(version)
docker run <image_nmae>:(version) => It will first check into our system for the image being pulled. If it is not found then it will be pulled and then started.

# 8) How to run a Docker container?

docker run <image_name>
docker run -d <image_name> : This command runs the container in detached mode. In this mode, we can use the terminal for any other operation without stopping the container.
docker run -p<host_port>:<container_port> <image_name> : Using this command, we are running the image by binding the host machine port with container's default port in order to avoid port conflict.
docker run -p7007:3600 mysql

# 9) How to start and stop docker container?
docker start <container_id>
docker stop <container_id>

# 10) How to check logs of a docker conatiner?
docker logs <container_id>/<container_name>
docker logs my_mysql

# 11) How to run docker container with a name?
docker run -d --name <custom_name> <image_name>

# 12) How to access container's terminal?
docker exec -it <container_id>/<container_name> /bin/bash(sh)
docker exec -it my_container /bin/bash

# 13) What is docker network?
Docker networking enables a user to link a Docker container to as many networks as he/she requires. Docker Networks are used to provide complete isolation for Docker containers.
Note: A user can add containers to more than one network.

docker network ls : This command lists all the docker networks.
docker network create <docker_network> : This command creates a new docker network.

# 14) How to run a docker image in a docker network?
docker run -d -p 27017:27017 --name <container_name> --net <network_name> <image_name>

# 15) What is docker-compose?
docker-compose is a structured way to contain docker commands of docker images.

docker run -d --name mongodb -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password --net mongo-network mongo 

docker-compose.yaml

version:'3'
services:
	mongodb:
	       image: mongo
               ports:
		  -27017:27017 (host:container)
               environment:
		  -MONGO_INITDB_ROOT_USERNAME=admin
		  -MONGO_INITDB_ROOT_PASSWORD=password
	<another>:
		------

When we use docker-compose, then there is not need of create a docker network because it will be created automatically.

# 16) How to use docker-compose?

docker-compose -f <docker-compose-filename> up
docker-compose -f mongo-docker-conpose.yaml up

docker-compose -f <docker-compose-filename> down

# 17) What is Dockerfile?
Blueprint for building images.

FROM node    => install node

ENV MONGO_DB_USERNAME=admin\
    MONGO_DB_PASSWORD=password

RUN mkdir -p /home/app   => create /home/app folder on container

COPY . /home/app         => copy current folder files from host to /home/app folder of container

CMD ["node","/home/app/server.js"] => run command node server.js on container

# 18) How to build an docker image from Dockerfile?

docker build -t myapp:1.0 .

Here . means our Dockerfile is in current directory

# 19) How to delete docker image?
docker rmi <image_name>

# 20) How to delete docker container?
docker rm <container_id>

# 21) When do we need docker volumes?
DockerVolumes are used for data persistance in docker container. Whenever we restart or remove the docker container, Data is gone.

# 22) What is docker volumes?
Host computer has physical file system but docker container has virtual file system. Using docker volume, we mount the file path on host system into the 
virtual file path of docker container. So, if something gets changed at the container file system that also gets reflected on the host file system and vice versa.
This way, data persistance is managed.

# Type of docker volumes -

i) Host volumes : we decides where on the host file system the reference is made 

docker run -v /home/mount/data:/var/lib/mysql/data

ii) Anonymous volume:
docker run -v /var/lib/mysql/data 
for each container a folder is automatically generated by docker that get mounted.

iii) Improved Anonymous volume(named volume): It should be used in production
docker run -v name:/var/lib/mysql/data

in docker-compose.yaml 

version:'3'
services:
	mongodb:
	       image: mongo
               ports:
		  -27017:27017 (host:container)
               environment:
		  -MONGO_INITDB_ROOT_USERNAME=admin
		  -MONGO_INITDB_ROOT_PASSWORD=password
	       vloumes:
	          - db-data:/var/lib/mysql/data

volumes:
    db-data

.....
.....

# The docker engine consists of three major components:

Docker Daemon: The daemon (dockerd) is a process that keeps running in the background and waits for commands from the client. The daemon is capable of managing various Docker objects.
Docker Client: The client  (docker) is a command-line interface program mostly responsible for transporting commands issued by users.
REST API: The REST API acts as a bridge between the daemon and the client. Any command issued using the client passes through the API to finally reach the daemon.
	
# Use multi-stage builds
With multi-stage builds, you use multiple FROM statements in your Dockerfile. Each FROM instruction can use a different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don’t want in the final image. To show how this works, let’s adapt the Dockerfile from the previous section to use multi-stage builds.

```	
# syntax=docker/dockerfile:1

FROM golang:1.16
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go ./
RUN CGO_ENABLED=0 go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=0 /go/src/github.com/alexellis/href-counter/app ./
CMD ["./app"]
```

# References links
https://www.freecodecamp.org/news/the-docker-handbook/
