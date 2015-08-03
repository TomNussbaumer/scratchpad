# Docker - Up and Running

## What is Docker?

> [Docker](www.docker.com) ia an open source containerization engine, which automates the packaging, shipping, and deployment of any software applications, that are presented as lightweight, portable, and self-sufficient containers, that will run virtually anywhere.
>
> **From:** [Learning Docker](http://www.amazon.com/Learning-Docker-Pethuru-Raj/dp/1784397938/)

In general a docker container contains an application/service with all it's dependencies like binaries, libraries, scripts, configuration files and so on. Beside the fact that it doesn't come with it's own kernel, it's essentially a complete linux system with it's own network interface.

[Docker](www.docker.com) puts software developers in charge of the complete server environment their application is running on. Due to the small scope of a container (usually one process per container) side effects from other stuff installed and running in the same scope are heavily reduced.

Docker consists of two major parts:

1. the docker engine to run containers
2. the docker image repository

Docker distinguishes between images and containers. Containers are based on images (the readonly part of their filesystem) and contain a temporary files which is layered above the readonly image to capture all changes.

At any point the base image and a container can be baked together to form a new image.

Due to the fact containers only holds the changes to the filesystem (deltas), containers itself are normally quite small and can be transfered between different environments within seconds.

If there are no changes to the filesystem you want to keep, you can delete the container once you have stopped it. This way you can treat your containers as **immutable server**.

Docker is based on [LXC (Linux Containers)](https://en.wikipedia.org/wiki/LXC).

## Installing, upgrading, deleting Docker

When you are running Ubuntu 14.04 like me installation, upgrading and deleting can be handled by the following one-liners.

**Installation or Upgrade**
``` 
$ wget -qO- https://get.docker.com/ | sh
```

**Uninstallation (Docker package only)**
``` 
$ sudo apt-get purge docker-engine
``` 

**Uninstallation (Docker + dependencies)**
``` 
$ sudo apt-get autoremove --purge docker-engine
``` 

The above commands will not remove images, containers, volumes, or user created configuration files on your host. If you wish to delete all images, containers, and volumes run the following command:

``` 
$ rm -rf /var/lib/docker
``` 

##Initial Configuration of Docker

To enable a user to run docker containers, the user must be part of the docker group. 

``` 
$ sudo usermod -aG docker name-of-user
``` 

Remember that you will have to log out and back in for this to take effect!

##Very first steps

```bash
# get version numbers of client and daemon
$ docker version

# start docker daemon
$ sudo service docker start

# get version numbers again
$ docker version

# show infos about local setup (no images and containers yet)
$docker info

# fetch global image, create container and run it
$docker run hello-world

# show all commands
$docker 
```

If you have followed my steps the output of the first command will contain an error message, because the docker daemon is not started yet.

**Note 1:** The hello-world image is an extreme example of a docker image, because it just contains a single statically linked binary (910 bytes in total).

**Note 2:** Each call of *docker run* produces a new container

**Note 2:** Why are there 2 images in the local repository with identical sizes?
