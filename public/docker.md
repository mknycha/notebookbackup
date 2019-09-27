## Commands

• `docker run -p 8080:3000 -v /var/www node` -> creates a data volume under /var/www (in the host mount), container will write stuff there.
  Useful when we just want to store volume data somewhere, but we do not want to access that data.

• `docker inspect mycontainer` -> shows stuff like volume location (under `Mounts/Destination`)

• `docker run -p 8080:3000 -v $(pwd):/var/www node` -> use current directory as the host mount (folder where I want my source code to be)

• `docker rm -v mycontainer` -> removes mycontainer along with the volume (omitting v flag does not remove the volume)

• `docker run -p 8080:3000 -v $(pwd):/var/www -w "/var/www" node npm start` -> apart from the above, run `npm start` in the directory `/var/www` inside the container

## Writing dockerfiles

```
FROM node:latest          # uses node latest image as a base

ENV NODE_ENV=production PORT=3000 # can be split into separate instructions

COPY  . /var/www          # copy entire content of current directory into /var/www in the container
WORKDIR /var/www          # set the context for different commands we want to run

VOLUME ["/var/www"]       # causes docker to mount this volume in directory on host filesystem (we can make this data stick around if even if container stopped)

RUN     npm install       # run stuff inside the container, in the workdir directory

EXPOSE 3000               # Exposes container on the port 3000. We can also go with EXPOSE $PORT, since 3000 is an env variable here

ENTRYPOINT ["npm", "start"] # Note the best way to go is write it as a JSON array
```
