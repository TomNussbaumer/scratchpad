# Project Tomagine

## Topic of this Project

As a demo project on how to use [Docker](www.docker.dom) in practice, I'm going to restructure www.tomagine.com to host more than just my old photoblog. Since this domain is running on a low-budget virtual instance I'm quite interested to see how much services I can run on it.

## Host environment

As a demo project for Docker the host environment should be as simple as possible. Essentially nothing but Docker and an SSH daemon should be installed. Until I'll dive into more sophisticated stuff like Docker Compose a few simple bash macros/scripts will handle the plumping of the various docker instances.

principles to follow for the docker instances:

* [KISS](en.wikipedia.org/wiki/KISS_principle)
* one process per instance (as long as practible)
* immutable servers (mutable data like logs should be written elsewhere)


## First steps

1. install host environment including Docker, SSH daemon, git and public ssh key for access
2. install a dockerized nginx instance as reverse proxy with its config outside of the instance
3. write macros to start/stop/reload proxy
4. dockerize old environment and write macros to start/stop it

## Additional Steps / Services (unordered list)

* Jekyll-based static blog as main page
* private github-like server (either gogs or gitea)
* private docker repository
* OpenID server (secrets server for docker instances?)

## Remarks

* central logging feature?
* monitoring health and metrics of single docker instances?
* web gui for administration?
* backend data store for docker registry? (S3, Azure, Google?)
* free SSH cert for tomagine.com?
