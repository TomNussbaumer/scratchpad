# Docker - Tips and Tricks

Docker is a quite fast running target. Make sure to use the latest stable release to profit from the enhanced features (like formatting the output of `docker ps`).

## Buildtime

### Using environment variables

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

### Piping through containers

```
## process log files by piping them through container

# variant 1: stdin and stdout wired automatically
cat /var/log/messages | docker run -i --rm busybox grep 'docker' | wc -l

## variant 2: only stdin wired. Fetch output from docker logs later
ID=$(cat /var/log/messages | docker run -i -a stdin busybox grep 'docker'); 
docker logs $ID | wc -l
docker rm $ID

```

### Filtering and formatting the output of `docker ps`

**Note:** formatting the output of `docker ps` was introduced in version 1.8.1. Check the output of `docker version` and upgrade if necessary. 

```



## format example (less columns)
##
## NOTE: you can guess the column names by examine the output of the normal 
##       'docker ps' command. Only column ID is special (.ID)
docker ps -a --format="table {{.Names}}\\t{{.Image}}\\t{{.Ports}}\\t{{.Status}}"

```

## Cleantime

```
## removing ALL containers 
docker rm $(docker ps -aq)       # variant 1
docker ps -aq | xargs docker rm  # variant 2

## removing untagged images
docker rmi $(docker images | grep “^<none>” | awk ‘{print $3}’)


