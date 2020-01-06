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




