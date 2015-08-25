# Docker - Tips and Tricks

Docker is a quite fast running target. Make sure to use the latest stable release to profit from the enhanced features (like formatting the output of `docker ps`).

## Buildtime

### Using Environment Variables

You can use environment variables which are specified in the same Dockerfile anywhere in it:

```
##-------------------------------------------------------
## silly demo
##
## will just list all files in /bin during 'docker build'
##-------------------------------------------------------
FROM busybox

# setup env vars
ENV DEMO_DIR="/bin" DEMO_CMD="ls -la"

# from here on you can use the env vars
WORKDIR $DEMO_DIR
RUN     $DEMO_CMD
```

### Volumes are Evil!

Well, no, they are not exactly evil per se, but you have to be very carefully to manage them. If you have a VOLUME line anywhere in your Dockerfile a volume will be automatically created each time a container will be created **unless** you specify to re-use a volume from an already existing container.

This behaviour in combination with the lacking/missing tooling for volumes may lead to many many dangling volumes you are not aware of filling up your disk.

IMHO you are much better off **NOT using keyword VOLUME at all**, but handle all volumes via commandline parameters of `docker create` and `docker run` with explicit host pathnames.

**NOTE 1:** automatically created volumes can be found in /var/lib/docker/volumes.
**NOTE 2:** Since Docker images are built on each other using the VOLUME keyword in a Dockerfile is even more evil. Always inspect image (`docker inspect an_image`) before usage and check for section `Volumes` to verify if it will automatically created volumes. The official nginx image, for example, does exactly that - trashing your disk without you knowing it ...

### Using a Docker image as build environment

Setting up build environments is painful. What tools are required? Are the tools on the building platform available? Behave the tools exactly the same (i.e. different versions or different brands)? And and and ...

With docker that burden is gone. Embed everything that's needed in a docker image and build everywhere without problems. 

An example of Docker as build environment which even does cross compiling for multiple platforms can be found [here](https://github.com/tianon/gosu).


## Runtime

### Piping through Containers

```
## process log files by piping them through container

# variant 1: stdin and stdout wired automatically
cat /var/log/messages | docker run -i --rm busybox grep 'docker' | wc -l

## variant 2: only stdin wired. Fetch output from docker logs later
ID=$(cat /var/log/messages | docker run -i -a stdin busybox grep 'docker'); 
docker logs $ID | wc -l
docker rm $ID
```

### Filtering and Formatting the Output of `docker ps`

**Note:** formatting the output of `docker ps` was introduced in version 1.8.1. Check the output of `docker version` and upgrade if necessary. 

```
## filtering examples
##
## -f id=12345
## -f label=akey - or -  label=akey=avalue
## -f name=substring   
## -f exited=1      (don't forget --all)
## -f status=type   (created|restarting|running|paused|exited)

docker ps -a -f name=ss   # show all with name containing 'ss'

## format example (less columns)
##
## NOTE: you can guess the column names by examine the output of the normal 
##       'docker ps' command. Only column ID is special (.ID)
docker ps -a --format="table {{.Names}}\\t{{.Image}}\\t{{.Ports}}\\t{{.Status}}"
```

### Share X11 directly

```
## variant 1 - share X11 socket directly
docker run -e DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix:ro ......

### variant 2 - share the .Xauthority file (READONLY)
# NOTE: must be mounted to the home directory of the container user
#       (in the example the homedir of root)
docker run -e DISPLAY -v $HOME/.Xauthority:/root/.Xauthority:ro ......
```

### Disable Security Features

Normally the host's security settings prevents running tools like `strace` in a container.

```
## variant 1 (specify exactly what should be enabled)
## ... this is enough to run strace ...
docker run --cap-add SYS_PTRACE --security-opt apparmor:unconfined .... 

## variant 2 (disable all security measures)
docker run --privileged ....
```

### Controlling Docker from within a Container

To control the docker daemon on the host from within a container you'll need to bind-mount the docker socket to the container.

```
## mount the docker daemon socket to the container
docker run -v /var/run/docker.sock:/var/run/docker.sock ....
```

### Get Docker Host Address from within a Container

To get the Docker host address on the private docker network from within a container:

```shell
## returns gateway (== Docker host address)
netstat -nr | grep '^0\.0\.0\.0' | awk '{print $2}'
```

### Get list of mounts within a Container

```shell
## NOTE: there are always at least 3 mounts within a container
##       /etc/resolv.conf, /etc/hostname and /etc/hosts
cat /proc/mounts | grep '^/dev/'

## EXAMPLE

docker run --name=data0 -v /sysdata -v /syscfg busybox sh -c "cat /proc/mounts | grep '^/dev/'"
docker rm -v data0
```

### Get own Name and IP of a Container

That's NOT an easy task, because it depends completely on how the container and its net infrastructure is configurated. There are various ways to get the container IP, but the least image depended way (what's installed) is to parse /etc/hosts

```shell
## if hostname is not configurated it is set to the 12-digit container id
my_ip=$(cat /etc/hosts | grep $(hostname) | cut -f1)
my_names=$(cat /etc/hosts | grep "$my_ip" | cut -f2)

## my_names will normally contain 3 names: the 12-digit id, a name and the
## name with postfix '.bridge'.
## NOTE: if a hostname is configurated, you will no longer have access to the
##       12-digit container id from within the container!
```

## Cleantime

```
## removing ALL stopped containers (don't do this in production, of course)
docker rm $(docker ps -aq)       # variant 1
docker ps -aq | xargs docker rm  # variant 2

## removing ALL stopped container and ALL of their automatically created VOLUMES
docker rm -v $(docker ps -aq)       # variant 1
docker ps -aq | xargs docker rm -v  # variant 2

## removing untagged/unneeded images
docker rmi $(docker images | grep “^<none>” | awk ‘{print $3}’) # variant 1
docker rmi $(docker images -aqf dangling=true)                  # variant 2
``
