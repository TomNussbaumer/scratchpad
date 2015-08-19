# Docker - Tips and Tricks

Docker is a quite fast running target. Make sure to use the latest stable release to profit from the enhanced features (like formatting the output of `docker ps`).

## Buildtime

### Using Environment Variables

You can use environment variables anywhere in a Dockerfile:

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

## Cleantime

```
## removing ALL stopped containers 
docker rm $(docker ps -aq)       # variant 1
docker ps -aq | xargs docker rm  # variant 2

## removing untagged/unneeded images
docker rmi $(docker images | grep “^<none>” | awk ‘{print $3}’) # variant 1
docker rmi $(docker images -aqf dangling=true)                  # variant 2
``

