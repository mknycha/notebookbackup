• `docker run -p 8080:3000 -v /var/www node` -> creates a data volume under /var/www (in the host mount), container will write stuff there.
  Useful when we just want to store volume data somewhere, but we do not want to access that data.

• `docker inspect mycontainer` -> shows stuff like volume location (under `Mounts/Destination`)

• `docker run -p 8080:3000 -v $(pwd):/var/www node` -> use current directory as the host mount (folder where I want my source code to be)

• `docker rm -v mycontainer` -> removes mycontainer along with the volume (omitting v flag does not remove the volume)

• `docker run -p 8080:3000 -v $(pwd):/var/www -w "/var/www" node npm start` -> apart from the above, run `npm start` in the directory `/var/www` inside the container
