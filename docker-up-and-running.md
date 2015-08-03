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

Docker images are strictly readonly. To enable writing for the runtime, a temporary filesystem is layered upon the readonly image which captures all changes. At any point this temporary filesystem can be committed as own image (holding a reference to the base image). Due to the fact that this filesystem holds only changes (deltas), the new image is normally quite small and can be transfered between different environments within seconds.

As far as I know there is no limit on how much images can be layered this way for a single running container. Nevertheless it's a good idea to collapse the image stack into a single image once in a while to prevent performance degration. 

If there are no changes you want to keep since the container has started, you can also delete the temporary filesystem once you have stopped the container. This way you can treat your container as **immutable server** (which is a very good idea for various reasons).
