# docker-images

# List of Useful Docker Images and their related Links

`docker search` ... hmmmmmmm ... it's a joke, isn't it? About 20 characters as image description and no possibility to get more informations about a single image. Forget about it.

The only useful source for informations about available Docker images is [the Docker Hub](https://hub.docker.com). There are already tons of images (20.000?), but not many of them are really helpful or even trustable.

First of all: there are [the Official Repositories](https://hub.docker.com/explore/). The naming may be a little bit missleading, because 'official' means here, these repositories are maintained by the people behind Docker and NOT the upstream maintainers of the corresponding software packages themself.

Nevertheless: You can always examine the corresponding Github repository and check the steps performed to build the images and/or build it by your own. For linux distributions like Ubuntu almost all what they do is importing an official released tarball and some additional configurations necessary for the docker environment.

Beside the official repositories here are image vendors I really like:

  * [Images by Phusion](https://hub.docker.com/search/?q=phusion&starCount=1)
  * [Images by CenturyLink](https://hub.docker.com/search/?q=centurylink&starCount=1)

**Ongoing list of useful images:**

link                                          | description
--------------------------------------------- | ---------------------------------------------
[busybox](https://hub.docker.com/_/busybox/)  | if you like it small ...
[ubuntu](https://hub.docker.com/_/ubuntu/)    | my favorite distro
[phusion/baseimage](https://hub.docker.com/r/phusion/baseimage/) | especially the docs about what they solved and why
[buildpack-deps](https://hub.docker.com/_/buildpack-deps/) | complete linux build environment
[centurylink/golang-builder](https://hub.docker.com/r/centurylink/golang-builder/) | go build :)


