# Docker Crash Course
https://www.youtube.com/playlist?list=PL4cUxeGkcC9hxjeEtdHFNYMtCpjNBm3h7

## Overview

- Docker

## Course Outline

1. What is Docker
2. Installing Docker
3. Images & Containers
4. Parent Images & Docker Hub
5. The Dockerfile
6. dockerignore
7. Starting and Stopping Containers
8. Layer Caching
9. Managing Images and Containers
10. Volumes
11. Docker Compose
12. Dockerizing a React App
13. Sharing Images on Docker Hub


## 1. What is Docker

Container

Can have many different projects with different dependencies

#Virtual machines: Has it's own full operating systems and typically slower
#Containers: share the host's operating system and typically quicker

## 2. Installing Docker

https://docs.docker.com/get-docker/


## 3. Images and Containers

Images:
- Like blueprints for containers
-  Runtime environment
-  Appliation code
-  Any dependencies
-  Extra configuration
-  Commands

Containers:
- Running an instance of an image
- Runs our application
- Isolated Process


## 4. Parent Images and Docker Hub

Images are made up from several layers:

1. Parent image: includes the OS and sometimes the runtime environment
2. Source code
3. dependencies

Docker Hub: 
- online repo of docker images
- containes premade parent images
Example:
For node: search for node parent image
### From docker hub:
- put this in terminal (PowerShell)
```
docker pull node
``` 
- can add tags for distributions 
> if we don't specify the default will be pulled
- After it is downloaded you can see the image in your docker desktop

### This image can be run:
- do nothing with additional options
- container is created for node:
- can `enter cli` `stop` `restart` `delete` 






## 5. The Dockerfile
API Folder
- `app.js` (has simple express app listening on port 4000)
- has dependencies in `package.json`

### Normally we would write
```
npm install
npm run
```

To run inside container --> We need an image

Firstly install the docker extension in vs code
Extensions --> Docker --> Install

Create `Dockerfile` (set of instructions to tell docker what to do)
```
FROM node:17-alpine #pulled from docker repo (it downloads if it doesn't exist)

WORKDIR /app #this tells docker that commands after this is done from this working directory

COPY . .
#COPY 
#"relative path" since they are in the same file as docker it is a .
#"path inside image to copy files to" . here means copy into the root 
#directory of the image we're creating

RUN npm install

EXPOSE 4000 #for port that we listen on (optional)

CMD  ["node", "app.js"] 
#CMD allows us to specify any commands that should be run in runtime after container starts to run
```

For building the dockerfile write in terminal
```
docker build -t myapp .
```
Can now see myapp in Docker Desktop



## 6. dockerignore
file called `.dockerignore`
has 
```
node_modules
*.md
```
means docker will ignore node_modules and any file with extension .md


## 7. Starting and Stopping Containers

###RUN FROM DOCKER DESKTOP
When we run the container we get options
- Container Name: myapp1_c
- Port: 4000 #Local host port vs container port don't have to be the same
RUN (green icon)
> Open in browser
We can see the JSON data ✔

### RUN FROM TERMINAL
```
docker images
```
Lists all imags
use repo name
```
docker run --name myapp_c1 myapp
```
> Open in browser
Doesn't work ❌

> Stop container
1- Open new terminal
2- 
```
docker ps #shows running containers
```
3-
```
docker stop myapp_c1
```
4- In older terminal you can see the process is stopped

> Run new container with port
```
docker run --name myapp_c2 -p 4000:4000 myapp
```
You can run in detached mode `-d` so we can use terminal later
> Open in browser
We can see the JSON data ✔

### STOP CONTAINER
```
docker stop myapp_c2
```
### RE-START CONTAINER
```
docker start myapp_c2
```




## 8. Layer Caching
docker does caching by itself
but 
after `COPY` things are done again making npm install run again
so we can `COPY package.json .` So are able to run `npm install` before `COPY . .`
that way we are able to cache `npm install` and not have to run it agian

## 9. Managing Images and Containers
### How to DELETE Images and Containers
`docker images` shows all images
`docker ps` shows running containers
`docker ps -a` shows all containers

### DELETE IMAGE
DELETE `docker image rm myapp4`
> Successfully deletes myapp4

DELETE `docker image rm myapp`
> Can't remove because it is used by a container

DELETE `docker image rm myapp5 -f` to force remove image used by container

### DELETE CONTAINER
> myapp_c2 uses myapp
`docker container rm myapp_c2`
> Successfully deletes container myapp_c2
> Now we are able to delete previously used image

### DELETE MULTIPLE CONTAINERS
`docker container rm myapp5_c myapp1_c`

### MANAGING IMAGES
Firstly delete all containers and images `docker system prune -a`
> Confirm
#### Creating a new image with a tag
`docker build -t myapp:v1`
`docker run --name myapp_c -p 4000:4000 myapp:v1`

## 10. Volumes
`docker start` vs `docker run`
> run makes a new container while start runs a container that already exists
> start doesn't pick up on changes even after restart as that container already ran (build)
> to see changes we need a new container which takes time

### TO SOLVE THIS WE MAKE VOLUMES
We can map changes with volumes to containers so that when we change something the container changes as well
> We can map the whole project

1. First install nodemon 
```
RUN npm install -g nodemon
.
.
.
CMD ["npm", "run", "dev"]
```
> This will run dev so that nodemon would detect changes

2. make new image `docker build -t myapp:nodemon .`
3. `docker run --name myapp_c_nodemon -p 4000:4000 --rm myapp:nodemon` rm to remove or delete container when we stop it later `--rm`
4. Change anything in file and save
> nothing changes
5. Make a volume ->
  - `docker stop myapp_c_nodemon` stop this one first
  - `docker run --name myapp_c_nodemon -p 4000:4000 --rm -v (absolute path):/app myapp:nodemon`
  - Now changes propagate, but also node modules is propagated even though we don't need it except in the container
  - We can solve this by adding an anonymous volume `-v /app/node_modules` content here presists
6. Run again and everything works
7. Change anything in file and save
> nodemon restarted and after refresh you see new change data



## 11. Docker Compose
to run multiple containers at once
> has to be in root dir

file name `docker-compose.yaml`
Instructions
```
version: "3.8"
services: 
  api:
    build: ./api
    container_name: api_c
    ports:
      - '4000:4000'
    volumes:
      - ./api:/app
      - ./app/node_modules
```

We can look at this yaml file as the command prompt steps
Now open terminal where docker-compose.yaml file is
`docker-compose up` when we do that docker is going to find it and work it out

to stop container `docker-compose down --rmi all -v` stops and deletes container
> rmi do delete images and v to delete volumes

## 12. Dockerizing a React App
we have `api` folder with node
we have `myblog` folder with react

inside root folder `myblog` of react
1. make `.dockerignore` and add node_modules there
2. make `Dockerfile`
3. Code for React App
```
FROM node:17-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

4. Now we configure docker-compose.yaml
```
.
.
.
services:
  .
  .
  .
  myblog:
    build: ./myblog
    container_name: myblog_c
    ports:
      - '3000:3000'
    stdin_open: true
    tty: true   --these two commands makes container start in interactive mode for react apps
```
5. Then run in terminal

## 13. Sharing Images on Docker Hub

1. Sign up to Docker Hub
2. Create Repository
3. Call it "myapi"
4. Add description
5. Choose Private or Public
6. To use it in CLI 
  - `docker login`
  - `docker push (name)/myapi`
  - `docker build -t (name)/myapi .`
> Last pushed a few seconds ago
7. To pull `docker pull (name)/myapi`
