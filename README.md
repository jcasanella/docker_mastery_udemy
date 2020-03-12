# Docker Mastery Udemy

Here you will find all the exercises from the Udemy Course: *Docker Mastery: with kubernetes + Swarm from a Docker Captain* (https://www.udemy.com/course/docker-mastery/). This course explains really clearly all the docker secrets with videos and some exercies. The course has a really good structure.

## Exercise 2

In this exercise, we must run at the same time the following services: 

* nginx: listen port 80:80
* mysql: listen port 3306:3306 and use variable MYSQL_RANDOM_ROOT_PASSWORD=yes, in order to create a random password
* httpd (apache server): listen port 8080:80


All of them need to run in the background (detached) and must assign a name. Check if all the containers are up. Once is done, remember to clean up everything.

### Run nginx container

Pull and run the container in detach mode and listening port 80:80

```
docker container run -p 80:80 -d --name nginx nginx:latest
```

In order to check if a container is running we can use any of these commands:

```
docker container top nginx
docker container inspect nginx
docker container stats nginx
```

Now, lets check everything is working fine and the logs created in the container, open a browser and use the following address: <localhost:80>. 

```
docker container logs nginx
```

Every time that we refresh the browser, will turn up a new entry in the log.

### Run httpd container

Pull and run the container in detach mode and listening port 8080:80

```
docker container run -p 8080:80 --name httpd -d httpd:latest
```

Like we did in nginx, we open a browser but this time will use the following address <localhost:8080>. Note the port in this case is different. We can check the logs of the container using:

```
docker container logs httpd
```

### Run mysql container

Pull and run the image

```
docker container run -p 3306:3306 -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql:latest
```

Once the server is up, we need to look for the password into the logs. Note if you use linux or mac, replace findstr for grep.

```
docker logs mysql | findstr -i GENERATED
```

With the password, we can connect to the server:

```
docker exec -it mysql mysql -u root -p (connect to the container)
```

In case, we're interested to connect to mysql using a mysql client, will run the following commands from mysql server.

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'jenni135';
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'jenni135';
```

### Clean up the containers and drop the images

```
docker container stop nginx httpd mysql
docker image rm --force httpd nginx mysql
```

## Exercise 3a

Use two different terminal windows to start bash in both *centos:7* and *ubuntu:14.04*, install on both containers curl and check the installed version. Remember to clean up both containers once is finished the exercise. This exercise will help us to understand how to connect inside a container. The connection is without ssh. The argument -it opens an interactive terminal.

```
docker container run --rm -it centos:7 bash
```

once we're inside the docker container we can install the required libraries.

```
yum update curl
curl --version
```

Lets do the same in that case for the ubuntu container.

```
docker container run --rm -it ubuntu:14.04 bash
apt-get update && apt-get install -y curl
curl --version
```

## Exercise 3b

The objective of this exercise is to create a "Poor Load Balancer" using a Round Robin DNS. The steps are:

* Create a new virtual network (default bridge driver)
* Run two containers at the same time, using the elasticsearch:2 image and associate to the network created in the previous step and to the same DNS (*net-alias*)
* Check if the DNS works. 
* Run curl several times, using the dns address, this will show you the containers created at the beginning of this activity.


### Create the virtual network

```
docker network create my_app_network
```

### Run the 2 containers at the same time, using elasticsearch:2 image

It's important to attach to the network created in the previous step and assign a network alias to be used as a DNS.

```
docker container run -d --network my_app_network --net-alias search elasticsearch:2
docker container run -d --network my_app_network --net-alias search elasticsearch:2
```

### Check if the DNS works

```
docker container run --rm --network my_app_network alpine  nslookup search
```

### Run curl using the DNS:port

```
docker container run --rm --network my_app_network centos curl -s search:9200
```

### Clean up the containers and network

```
docker container rm -f 3c88 b46c
docker network rm my_app_network
```

## Exercise 4

[Create a Dockerfile](exercise4/README.md)

Sometimes need to unblind the ports, run the following commands:
```
docker system prune
docker volume prune
docker network prune
```

## How to persist data

Containers are designed to be immutable and temporal, and to separate binaries from data. In order to use data we must use **volumes**.

* First option, specify in the Dockerfile, so that means if we dont want to keep for more time the volumes, we must drop it manually. (Entry in the Dockerfile: **VOLUME /location**)

Problem: By default is not using a friendly name, we can check with:
```
docker container inspect container_name 
docker volume ls
```

To use a friendly name, we must run the container as:
```
docker container run -d --name ... -v alias_to_use:location image_id
```

Other option to persist data is creating the volume before to run the container:

```
docker volume create --name nexus-data
docker container run -d -p 8081:8081 --name nexus -v volume-local:/nexus-data sonatype/nexus
```

This creates in the container a folder called nexus-data and persists the data into the volume-local. In windows is //d/folder
Maybe you need to enable from the docker desktop the unit d to share

* **docker container run -d -v /Folder/host:/path_container docker_image** (linux/mac)
* **docker container run -d -v //unit/Folder/host:/path_container docker_image** (windows)

## Exercise 5a

Let's suppose we need to upgrade a database, for this exercise Postgres. We know, following the best practices in containers,
we should use the image with a new version instead of patching but what about if we can't replace the container and we need to 
upgrade the database.

* Create a **Postgres** container with named volume psql-data using version **9.6.1**
* Use **Docker Hub** to learn **Volume path** and versions needed to run it (check Dockerfile)
* Check the logs to see when it's finished creating the databases, will be one
point that the logs will stop. 
* If it's done correctly, run **docker volume ls** you will see the volume.
* Stop the container and create a new one, using the version **9.6.2** and the same named volume.
* Check again the logs

**Solution:**

 Look in Docker Hub for Postgres 9.6.1 and look for the Volume entry in the Dockerfile. The volume is: **/var/lib/postgresql/data**

```
docker pull postgres:9.6.1
docker volume create psql-data
docker container run --name postgres -v psql-data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=mysecretpassword -d postgres:9.6.1
docker logs postgres
docker volume ls
docker container inspect postgres
docker container stop postgres
docker container rm postgres
docker container run --name postgres -v psql-data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=mysecretpassword -d postgres:9.6.2
docker logs postgres
```