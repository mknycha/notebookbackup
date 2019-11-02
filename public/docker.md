Docker containers creation:
- Create Dockerfile - defines what goes on in your containers. Add here access to resources from the 'outside world', like files of your app and requirements.
- Specify the requirements (libraries) in a separate file and include it as per the above.
- Run Build - create docker image. Worth adding a tag (descriptive name) when creating it.
- Run the app. To access it locally, map the port of your local machine to the one exposed in dockerfile.
- Share the image to registry. Registry -> Collection of repos. Repo -> Collection of images.
	- You can create a docker account to use docker public registry.
- Connect to docker account. Now whenever you run image locally, it will be pulled if not found.

## Commands

• `docker run -p 8080:3000 -v /var/www node` -> creates a data volume under /var/www (in the host mount), container will write stuff there.
  Useful when we just want to store volume data somewhere, but we do not want to access that data.

• `docker inspect mycontainer` -> shows stuff like volume location (under `Mounts/Destination`)

• `docker run -p 8080:3000 -v $(pwd):/var/www node` -> use current directory as the host mount (folder where I want my source code to be)

• `docker rm -v mycontainer` -> removes mycontainer along with the volume (omitting v flag does not remove the volume)

- `docker run -p 8080:3000 -v $(pwd):/var/www -w "/var/www" node npm start` -> apart from the above, run `npm start` in the directory `/var/www` inside the container
  - `-d` - run in detached mode (gives back the control to the console)
  - `-it` - run in an interactive mode
  - `--link my-mysql-name:mysql` - link container named `my-mysql-name` to `mysql` within that container (legacy linking)

• `docker build -t <your username>/node .` -> builds an image with a tag provided based on Dockerfile. `.` is the build context

• `docker build -f Dockerfile -t tag .` -> builds an image based on the dockerfile passed under f flag

• `docker rm -f $(docker ps -a -q)` -> force removing quitely all the images

## Writing dockerfiles

```
FROM node:latest          # uses node latest image as a base

LABEL Name=sth            # Label for the image. You can use labels to organize your images, record licensing information, annotate relationships between containers, volumes, and networks, or in any way that makes sense for your business or application.

ARG source=.              # Defines an argument with an optional default value that can overridden by the user when the container is run.

ENV NODE_ENV=production PORT=3000 # can be split into separate instructions

COPY  . /var/www          # copy entire content of current directory into /var/www in the container
WORKDIR /var/www          # set the context for different commands we want to run

VOLUME ["/var/www"]       # causes docker to mount this volume in directory on host filesystem (we can make this data stick around if even if container stopped). In this particular file it may cause issues since the volume is mount when container is run, while npm install will be run earlier and not used.

RUN     npm install       # run stuff inside the container, in the workdir directory

EXPOSE 3000               # Exposes container on the port 3000. We can also go with EXPOSE $PORT, since 3000 is an env variable here

ENTRYPOINT ["npm", "start"] # Note the best way to go is write it as a JSON array
```

## Running containers in a network (linking)

Cockroach db example.
Let's create a bridge and run several nodes.
```
docker network create -d bridge roachnet
```
```
docker run -d --name=roach1 --hostname=roach1 --net=roachnet -p 26257:26257 -p 8080:8080  -v "${PWD}/cockroach-data/roach1:/cockroach/cockroach-data"  cockroachdb/cockroach:v19.1.5 start --insecure
```
```
docker run -d --name=roach2 --hostname=roach2 --net=roachnet -v "${PWD}/cockroach-data/roach2:/cockroach/cockroach-data" cockroachdb/cockroach:v19.1.5 start --insecure --join=roach1
```
```
docker run -d --name=roach3 --hostname=roach3 --net=roachnet -v "${PWD}/cockroach-data/roach3:/cockroach/cockroach-data" cockroachdb/cockroach:v19.1.5 start --insecure --join=roach1
```
Now we can run the container in the same network:
```
docker run --net=roachnet my-docker-image
```

## Docker compose
- docker-compose.yml -> defines all the services. The service is something to we run in a container - PHP, node.js, redis etc.
- build process generates the images (services) based on docker-compose.yml
- configuration options:
  - build - build context, docker file used etc.
  - environment variables
  - image
  - networks - define networks to link things up
  - ports
  - volumes
- commands:
  - `build` -> builds the images. Use `build service_name` to build only paritulcar service 
  - `up` -> start the containers
  - `down` -> take the containers down
  - `logs`
  - `ps`
  - `rm`
  

## docker-compose.yml example
This file runs all the above cockroach nodes at once, and wihtin a network `tradingbot_roachnet`
```
version: '2.1'

services:
  roach1:
    container_name: roach1
    image: cockroachdb/cockroach:v19.1.5
    networks:
      - roachnet
    ports:
      - "26257:26257"
      - "8080:8080"
    volumes:
      - "${PWD}/cockroach-data/roach1:/cockroach/cockroach-data"
    command: start --insecure
  roach2:
    container_name: roach2
    image: cockroachdb/cockroach:v19.1.5
    networks:
      - roachnet
    volumes:
      - "${PWD}/cockroach-data/roach2:/cockroach/cockroach-data"
    command: start --insecure --join=roach1
  roach3:
    container_name: roach3
    image: cockroachdb/cockroach:v19.1.5
    networks:
      - roachnet
    volumes:
      - "${PWD}/cockroach-data/roach3:/cockroach/cockroach-data"
    command: start --insecure --join=roach1

networks:
  roachnet:
    driver: bridge
```
