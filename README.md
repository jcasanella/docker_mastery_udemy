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

## Exercise 5b (Bind mounts)

Use a Jekyll "Static Site Generator" to start a local web server. Objective is to mount a host folder inside the docker container. This
creates an association using a folder between the host and the container. Any change done in the host folder will be viewed in the docker
container. 

* Mount the folder: [exercise5b](exercise5b)
* Edit the file with your favourite editor, container detects changes with host files and updates web server
* Start container 
```
docker container run -p 80:4000 -v ${pwd}:/site bretfisher/jekyll-serve
```
* Change the file in _posts\ and refresh the browser to see changes. Check the logs inside the docker container

```
cd exercise5b
docker container run -p 80:4000 -v ${pwd}:/site bretfisher/jekyll-serve -d
docker logs flamboyant_murdock
```

## Exercise 6a

Build a basic compose file for a 
* **Drupal** content management system website. Look in Docker Hub. 
* Use the **Drupal** image along with the **Postgres** image. 
* Expose port **8080** for Drupal (localhost:8080).
* For the Postgres set up the **POSTGRES_PASSWORD**. 
* Create a volume for **Drupal**, check Docker Hub documentation.

Once is done, set up Drupal from
the browser, Drupal assumes Db is localhost, but there's an advanced option
to change it. 

[Solution](exercise6a) 

## Exercise 6b

Purpose: Build a custom Drupal image (Dockerimage) and start up everything with a
```docker-compose up``` including a Db to store the changes and volumes to remember
across compose restarts. Start with the compose file from the previous exercise as starting point. 

### Dockerfile

* First you need to build a custom Dockerfile using Drupal version 8.8.2
* Run apt package manager to install git. 
* Remember to clean up everything after your apt install with 
```rm -rf /var/lib/apt/lists/*``` See in Drupal Docker Hub.
* Change the **WORKDIR** to **/var/www/html/themes**
* Use git to clone the theme from [https://git.drupal.org/project/bootstrap.git]https://git.drupal.org/project/bootstrap.git 
use branch **8.x-3.x**
* Change permissions on files and don't want to use another image. The drupal container runs as
**www-data** user but the build actually runs as **root**. Use **chown** to change the owner of the files.
* Change the work directory to its default.  (**/var/www/html**)

### Compose File 

* Build custom image for this compose fro Drupal service. Use commpose file from previous exercise as 
starting point. 
* Rename image to *custom-drupal* as we want to make a new image based on the official *drupal:8.8.2*
* Remember to use the Dockerfile to be build from the Docker compose instead of pulling from Docker hub.
* For the `postgres:12.1` service, you need the same password as in previous assignment, but also add a volume 
for `drupal-data:/var/lib/postgresql/data` so the database will persist across Compose restarts

### How to test it

* Start containers like before, configure Drupal web install like before.
* After website comes up, click on `Appearance` in top bar, and notice a new theme called `Bootstrap` is there. 
That's the one we added with our custom Dockerfile.
* Click `Install and set as default`. Then click `Back to site` (in top left) and the website interface should look 
different. You've successfully installed and activated a new theme in your own custom image. 
* If you exit (ctrl-c) and then `docker-compose down` it will delete containers, but not the volumes, so on
next `docker-compose up` everything will be as it was.


Docker file first, that is a custom image of a Drupal, with a template added. 
Note: Dockerfile is a custom image for Drupal. This Dockerimage will build with a bootstrap template.
Use Drupal image  along with the postgres image as before.  

[Solution](exercise6b)

## Docker Swarm

### Basics

By default swarm is not enabled. We can see the swarm status running: `docker info` and looking for the swarm entry. Let's see how to enable and disable the swarm:

```
docker swarm init
docker swarm leave
```

We can check how many swarms nodes are running with

```
docker node ls

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
13wzrf9msxr8hdq8v4r1y9td7 *   docker-desktop      Ready               Active              Leader              19.03.8
```

To enable a service with one instance and list the instances running

```
docker service create alpine ping 8.8.8.8
docker service ls

ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
r9xtmnd5dzg3        quirky_montalcini   replicated          1/1                 alpine:latest
```

In this example we can see, how many replicas we have. Swarm will try out all the time to match number of replicas with the desired replicas.

To check the status of any instance used with the service,  run the following command:

```
docker service ps r9xtmnd5dzg3

ID                  NAME                  IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
jzv7optzwgla        quirky_montalcini.1   alpine:latest       docker-desktop      Running             Running 2 minutes ago
```

Now lets add some nodes to our service. Run the following command:

```
docker service update r9xtmnd5dzg3 --replicas 3
docker service ls

ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
r9xtmnd5dzg3        quirky_montalcini   replicated          3/3                 alpine:latest
```

There're 3 replicas running, if one of the replicas is down, Swarm will manager to run the requested number of replicas. 

```
docker service ps r9xtmnd5dzg3

ID                  NAME                  IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
jzv7optzwgla        quirky_montalcini.1   alpine:latest       docker-desktop      Running             Running 23 hours ago
vxmx0v05nqai        quirky_montalcini.2   alpine:latest       docker-desktop      Running             Running 8 minutes ago
nqvrqgn6igqm        quirky_montalcini.3   alpine:latest       docker-desktop      Running             Running 8 minutes ago

docker container ls

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
9de78f9100e6        alpine:latest       "ping 8.8.8.8"      19 minutes ago      Up 19 minutes                           quirky_montalcini.2.vxmx0v05nqaiz7v5t5a4fr71z
5b9f664aea3b        alpine:latest       "ping 8.8.8.8"      19 minutes ago      Up 19 minutes                           quirky_montalcini.3.nqvrqgn6igqmhstylq7ptxt8j
e2d207b6deb3        alpine:latest       "ping 8.8.8.8"      23 hours ago        Up 23 hours                             quirky_montalcini.1.jzv7optzwglawqaill7tx9pwc

docker container rm -f 9de78f9100e6  
docker service ps r9xtmnd5dzg3

ID                  NAME                      IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR                         PORTS
jzv7optzwgla        quirky_montalcini.1       alpine:latest       docker-desktop      Running             Running 23 hours ago
p7p4rp46dhl0        quirky_montalcini.2       alpine:latest       docker-desktop      Running             Running 1 second ago
vxmx0v05nqai         \_ quirky_montalcini.2   alpine:latest       docker-desktop      Shutdown            Failed 6 seconds ago     "task: non-zero exit (137)"
nqvrqgn6igqm        quirky_montalcini.3       alpine:latest       docker-desktop      Running             Running 21 minutes ago

docker service ls

ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
r9xtmnd5dzg3        quirky_montalcini   replicated          3/3                 alpine:latest
```

After drop one of the instances, Swarm started another instance in order to have the requested number of replicas.