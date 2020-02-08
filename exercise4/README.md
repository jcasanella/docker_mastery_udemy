# Create a Dockerfile

This is an exercise to understand the basics of Dockerfile

## Description of the exercise

This dir contains a Node.js app, you need to get it running in a container
No modifications to the app should be necessary, only create a **Dockerfile**

Folders to be included in the image:

* /bin
* /public
* /routes
* /views
* app.js
* package.json

## Overview of this assignment

Use the instructions from developer below to create a working Dockerfile. Steps to test the image:

* Once Dockerfile builds correctly, start container locally to make sure it works on http://localhost 
* Ensure image is named properly for your Docker Hub account with a new repo name push to Docker Hub. 
* Go to https://hub.docker.com and verify.
* Remove local image from cache.
* Start a new container from your Hub image, and watch how it auto downloads and runs.
* test again that it works at http://localhost


### Instructions from the app developer

These're all the steps to build the image:

1. Use the **'node'** official image, with the **alpine 6.x** branch.
2. This app must listen on port 3000, but the container should launch on port 80.
    2.1. So it will respond to http://localhost:80 on your computer
3. Use alpine package manager to install **tini**: ```apk add --update tini```
4. Create directory **/usr/src/app** for app files with ```mkdir -p /usr/src/app```
5. Node uses a "package manager", so it needs to copy in **package.json** file.
6. Run ```npm install``` to install dependencies from that file
7. To keep it clean and small, run ```npm cache clean --force``` 
8. Copy in all files from current directory
9. Start container with command ```/sbin/tini -- node ./bin/www```


In the end you should be using **FROM**, **RUN**, **WORKDIR**, **COPY**, **EXPOSE**, and **CMD** commands.

### How to build the image and run the container

```
docker build -t jcasanella/node .
docker container run --rm -p 80:3000 jcasanella/node
```

### Create the tag and push to Docker Hub

```
docker tag $id_image jcasanella/node:1.0.0
docker login --username=jcasanella
docker push jcasanella/node
```