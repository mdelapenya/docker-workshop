[![www.liferay.com](https://web.liferay.com/image/image_gallery?uuid=6f04350d-5873-48e2-88c8-200b48a77ad4&groupId=5912873&t=1325511154018)](https://www.liferay.com)

# Docker Workshop - Docker Basics

This section is separated in:

* [CLI Basics](#cli-basics)
* [Dockerfile basics](#dockerfile-basics)

# CLI Basics

### Version

Check you have latest version of docker installed:

```shell
docker version
```

* If you don't have docker installed, check [here](https://docs.docker.com/installation/#installation)
* If you're not on the latest version, it will prompt you to update
* If you're not on docker group you might need to prefix commands with `sudo`. See [here](http://docs.docker.com/installation/ubuntulinux/#giving-non-root-access) for details about it.

### Commands

Check the available docker commands

```shell
docker
```

* Whenever you don't remember a command, just type docker
* For more info, type `docker help COMMAND` (e.g. `docker help run`)
* If you want to use autocomplete capabilities in [Mac](http://superuser.com/questions/288438/bash-completion-for-commands-in-mac-os-x/288491#288491):

```shell
brew install bash-completion
curl -XGET https://raw.githubusercontent.com/docker/docker/master/contrib/completion/bash/docker > `brew --prefix`/etc/bash_completion.d/docker
```
Don't forget to add this to your .bashrc
```shell
if [ -f $(brew --prefix)/etc/bash_completion ]; then
  . $(brew --prefix)/etc/bash_completion
fi
```

### RUN a "Hello World" container

```shell
docker run alpine:3.3 echo "Hello World"
```

* If the Image is not cached, it pulls it automatically
* It prints `Hello World` and exits

### RUN an interactive Container

```shell
docker run -it alpine sh
  cat /etc/os-release
```

* **-i**: Keep stdin open even if not attached
* **-t**: Allocate a pseudo-tty

### RUN a Container with pipeline

```shell
cat /etc/resolv.conf | docker run -i alpine:3.3 wc -l
```

### SEARCH a Container

```shell
docker search -s 10 nginx
```

* **-s**: Only displays with at least x stars

### RUN a Container and expose a Port

On Linux:
```shell
docker run -d -p 4000:80 nginx
google-chrome localhost:4000
```

On Mac:
```shell
docker run -d -p 4000:80 nginx
open "http://localhost:4000"
```

On Windows:
```shell
docker run -d -p 4000:80 nginx
start chrome "http://localhost:4000"
```

* **-d**: Detached mode: Run container in the background, print new container id
* **-p**: Publish a container's port to the host (format: *hostPort:containerPort*)
* For more info about the container, see [nginx](https://registry.hub.docker.com/_/nginx/)

### RUN a Container with a Volume

NOTE: Make sure to be on `Docker Workshop` directory since we'll use volume mounts in the containers of directories of the repository.

On Linux:
```shell
docker run -d -p 4001:80 -v $(pwd)/code/hello-world/site/:/usr/share/nginx/html:ro nginx
google-chrome localhost:4001
```

On Mac:
```shell
docker run -d -p 4001:80 -v $(pwd)/code/hello-world/site/:/usr/share/nginx/html:ro nginx
open "http://localhost:4001"
```

On Windows:
```shell
docker run -d -p 4001:80 -v $(pwd)/code/hello-world/site/:/usr/share/nginx/html:ro nginx
start chrome "http://localhost:4001"
```

* **-v**: Bind mount a volume (e.g., from the host: -v /host:/container, from docker: -v /container)
* The volume is **linked** inside the container. Any external changes are visible directly inside the container.
* This example breaks the immutability of the container, good for debuging, not recommended for production (Volumes should be used for data, not code)

## Exercise 1 (10 mins)

* Build a static website
* Run it on your machine
* Share your (non-localhost) url with your buddies

# Dockerfile Basics

### BUILD a Git Client Container

Create a Git Container manually:

```shell
docker run -it --name git alpine sh
  apk --update add git
  git version
  exit
docker commit git docker-git
docker rm git
docker run --rm -it docker-git git version
docker rmi docker-git
```

* **--name**: Assign a name to the container
* **commit**: Create a new image from a container's changes
* **rm**: Remove one or more containers
* **rmi**: Remove one or more images
* **--rm**: Automatically remove the container when it exits

Create a Git Container with Dockerfile:

```shell
cd code/docker-git
docker build -t docker-git .
docker run -it docker-git git version
```

* **build**: Build an image from a Dockerfile

[code/docker-git/Dockerfile](../../code/docker-git/Dockerfile)
```
FROM alpine:3.3
RUN apk update
RUN apk add git
```

* The **FROM** instruction sets the Base Image for subsequent instructions
* The **RUN** instruction will execute any commands in a new layer on top of the current image and commit the results

### BUILD an Apache Server Container

Create an Apache Server Container with Dockerfile:

```shell
cd code/docker-apache2
docker build -t docker-apache2 .
docker run -d -p 4003:80 docker-apache2
```

On Linux:
```shell
google-chrome localhost:4003
```

On Mac:
```shell
open "http://localhost:4003"
```

On Windows:
```shell
start chrome "http://localhost:4003"
```

[code/docker-apache2/Dockerfile](../../code/docker-apache2/Dockerfile)
```
FROM alpine:3.3
RUN apk --update add apache2 && rm -rf /var/cache/apk/*
RUN mkdir -p /run/apache2
EXPOSE 80
CMD httpd -D FOREGROUND
```

* The **EXPOSE** instructions informs Docker that the container will listen on the specified network ports at runtime
* The **CMD** instruction sets the command to be executed when running the image

### BUILD a Static website Image

```shell
cd code/hello-world
docker build -t hello-world .
docker run -d --name hello -P hello-world
```

On Linux:
```shell
google-chrome $(docker port hello 80 | sed 's/0.0.0.0://g')
```

On Mac:
```shell
open "http://localhost:$(docker port hello 80 | sed 's/0.0.0.0://g')"
```

On Windows:
```shell
start chrome "http://localhost:$(docker port hello 80 | sed 's/0.0.0.0://g')"
```

* **-P**: Publish all exposed ports to the host interfaces
* **port**: Lookup the public-facing port that is NAT-ed to PRIVATE_PORT

[code/hello-world/Dockerfile](../../code/hello-world/Dockerfile)
```
FROM nginx:1.8-alpine
ADD site /usr/share/nginx/html
```

* The **ADD** instruction will copy new files from <src> and add them to the container's filesystem at path <dest>

## Exercise 2 (10 mins)

* Build your website with Dockerfile
* Run an instance
* Share your (non-localhost) url with your buddies

### PUSH Image to a Registry

For this step, we'll need to launch a registry:

```shell
docker run -d -p 5000:5000 --name registry registry:2
```

Then tag your image under the registry namespace and push it there:

```
REGISTRY=localhost:5000
docker tag hello-world $REGISTRY/$(whoami)/hello-world
docker push $REGISTRY/$(whoami)/hello-world
```

* **tag**: Tag an image into a repository
* **push**: Push an image or a repository to a Docker registry server

## Exercise 3 (10 mins)

* Push your website to the local Registry (use your github username)
* Push your website image
* Share your image name with your buddies

### PULL Image from a Repository

```shell
docker pull $REGISTRY/$(whoami)/hello-world
docker run -d -P --name=registry-hello $REGISTRY/$(whoami)/hello-world
```

On Linux:
```shell
google-chrome $(docker port registry-hello 80)
```

On Mac:
```shell
open "http://localhost:$(docker port registry-hello 80 | sed 's/0.0.0.0://g')"
```

On Windows:
```shell
start chrome "http://localhost:$(docker port hello 80 | sed 's/0.0.0.0://g')"
```

* **pull**: Pull an image or a repository from a Docker registry server


# Navigation

Previous | Next
:------- | ---:
← [Docker Workshop - Home](https://github.com/mdelapenya/docker-workshop) | [Docker Workshop - Docker machine](../01-docker-machine) →

# Credits

This workshop was prepared by [harbur.io](http://harbur.io), and adapted for Liferay's internal purpose by [Liferay, Inc.](https://www.liferay.com), under MIT License. Feel free to fork and improve.
