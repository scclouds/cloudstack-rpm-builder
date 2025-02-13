**Note: As the official repository seems unattended, we will continue the efforts in this one.**

# CloudStack RPM Package builder using Docker

[![Build Status](https://github.com/scclouds/cloudstack-rpm-builder/workflows/ci/badge.svg)](https://github.com/scclouds/cloudstack-rpm-builder/actions)
[![Docker Pulls](https://img.shields.io/docker/pulls/scclouds/cloudstack-rpm-builder.svg)](https://store.docker.com/community/images/scclouds/cloudstack-rpm-builder)
[![license](https://img.shields.io/github/license/scclouds/cloudstack-rpm-builder.svg)](https://github.com/scclouds/cloudstack-rpm-builder/blob/master/LICENSE)

Docker images for building Apache CloudStack RPM packages.

This will give portable, immutable and reproducable mechanism to build packages
for releases. A very good candidate to be used by the Jenkins slaves of the
project.

## Table of Contents

- [Supported tags and respective `Dockerfile` links](#supported-tags-and-respective-dockerfile-links)
- [Packages installed in container](#packages-installed-in-container)
- [Build RPM packages](#build-rpm-packages)
  - [Pull Docker images](#pull-docker-images)
  - [Build local repository](#build-local-repository)
    - [Clone Apache CloudStack source code](#clone-apache-cloudstack-source-code)
    - [Build packages of local repository](#build-packages-of-local-repository)
  - [Build remote repository](#build-remote-repository)
    - [Build packages of remote repository](#build-packages-of-remote-repository)
- [Building tips](#building-tips)
  - [Maven cache](#maven-cache)
  - [Adjust host owner permission](#adjust-host-owner-permission)
- [Builder help](#builder-help)
- [License](#license)

## Supported tags and respective `Dockerfile` links

- [`centos7-jdk11-gcc9.3`(centos7-jdk11-gcc9.3/Dockerfile)][centos7-jdk11-gcc9.3-dockerfile]
- [`centos7-jdk11`(centos7-jdk11/Dockerfile)][centos7-jdk11-dockerfile]
- [`centos7-jdk8`(centos7-jdk8/Dockerfile)][centos7-jdk8-dockerfile]
- [`centos6` (centos6/Dockerfile)][centos6-dockerfile] [EOL - not supported anymore]

## Packages installed in container

List of available packages inside the container:

- rpm-build
- yum-utils
- createrepo
- mkisofs
- git
- java (JDK 8 or JDK 11)
- maven 3.5
- tomcat
- python
- python3
- locate
- which
- nodejs

## Build RPM packages

Building RPM packages with the Docker container is rather easy, a few steps are
required:

### Pull Docker images

Let's assume we want to build packages for CentOS 7 on CentOS 7. We pull that
image first:

```bash
docker pull scclouds/cloudstack-rpm-builder:centos7-jdk11
```

You can replace `centos7-jdk11` tag by [one of the other tags].

### Build local repository

You can clone the CloudStack source code from repository locally on your machine
and build packages against that.

#### Clone Apache CloudStack source code

The first step required is to clone the CloudStack source code somewhere on the
filesystem, in `/tmp` for example:

```bash
git clone https://github.com/apache/cloudstack.git /tmp/cloudstack
```

Now that you have done so we can continue.

#### Build packages of local repository

Now that we have cloned the CloudStack source code locally, we can build packages
by mapping `/tmp` into `/mnt/build` in the container. (Note that the container
always expects the `cloudstack` code exists in `/mnt/build` path.)

```bash
docker run \
    -v /tmp:/mnt/build \
    scclouds/cloudstack-rpm-builder:centos7-jdk11 --distribution centos7 [ARGS...]
```

Or if your local cloudstack folder has other name, you need to map it to
`/mnt/build/cloudstack`.

```bash
docker run \
    -v /tmp/cloudstack-custom-name:/mnt/build/cloudstack \
    scclouds/cloudstack-rpm-builder:centos7-jdk11 --distribution centos7 [ARGS...]
```

After the build has finished the `.rpm` packages are available in
`/tmp/cloudstack/dist/rpmbuild/RPMS` on the host system.

### Build remote repository

Also you can build RPM packages of any remote repository without the need to manually
clone it first. You only need to specify git remote and git ref you intend to build
from.

#### Build packages of remote repository

Now let's assume we want to build packages of `HEAD` of `master` branch from
[https://github.com/apache/cloudstack] repository, we build packages by mapping
`/tmp` into `/mnt/build` in the container. The container will clone the repository
(defined by `--git-remote` flag) and check out the REF (defined by `--git-ref`
flag) in `/mnt/build/cloudstack` inside the container and can be accessed from
`/tmp/cloudstack` from the host machine.

```bash
docker run \
    -v /tmp:/mnt/build \
    scclouds/cloudstack-rpm-builder:centos7-jdk11 \
        --git-remote https://github.com/apache/cloudstack.git \
        --git-ref master \
        --distribution centos7 [ARGS...]
```

Note that any valid git Refspec is acceptable, such as:

- `refs/heads/<BRANCH>` to build specified Branch
- `<BRANCH>` short version of build specified Branch
- `refs/pull/<NUMBER>/head` to build specified GitHub Pull Request
- `refs/merge-requests/<NUMBER>/head` to build specified GitLab Merge Request
- `refs/tags/<NAME>` to build specified Tag

After the build has finished the `.rpm` packages are available in
`/tmp/cloudstack/dist/rpmbuild/RPMS` on the host system.

## Building tips

Check the following tips when using the builder:

### Maven cache

You can provide Maven cache folder (`~/.m2`) as a volume to the container to make
it run faster.

```bash
docker run \
    -v /tmp:/mnt/build \
    -v ~/.m2:/root/.m2 \
    scclouds/cloudstack-rpm-builder:centos7-jdk11 --distribution centos7 [ARGS...]
```

### Adjust host owner permission

Builder container in some cases (e.g. using `--use-timestamp` flag) may change
the file and directory owner shared from host to container (through volume) and
it will create `dist` directory which holds the final artifacts. You can provide
`USER_ID` (mandatory) and/or `USER_GID` (optional) from host to adjust the owner
from whitin the container.

This is specially useful if you want to use this image in Jenkins job and want to
clean up the workspace afterward. By adjusting the owner, you won't need to give
your Jenkins' user `sudo` privilege to clean up.

```bash
docker run \
    -v /tmp:/mnt/build \
    -e "USER_ID=$(id -u)" \
    -e "USER_GID=$(id -g)" \
    scclouds/cloudstack-rpm-builder:centos7-jdk11 --distribution centos7 [ARGS...]
```

## Builder help

To see all the available options you can pass to `docker run ...` command:

```bash
docker run \
    -v /tmp:/mnt/build \
    scclouds/cloudstack-rpm-builder:centos7-jdk11 --help
```

## License

Licensed under [Apache License version 2.0]. Please see the [LICENSE] file
included in the root directory of the source tree for extended license details.

[centos7-jdk11-gcc9.3-dockerfile]: https://github.com/scclouds/cloudstack-rpm-builder/blob/master/centos7-jdk11-gcc9.3/Dockerfile
[centos7-jdk11-dockerfile]: https://github.com/scclouds/cloudstack-rpm-builder/blob/master/centos7-jdk11/Dockerfile
[centos7-jdk8-dockerfile]: https://github.com/scclouds/cloudstack-rpm-builder/blob/master/centos7-jdk8/Dockerfile
[centos6-dockerfile]: https://github.com/scclouds/cloudstack-rpm-builder/blob/master/centos6/Dockerfile
[one of the other tags]: #supported-tags-and-respective-dockerfile-links
[https://github.com/apache/cloudstack]: https://github.com/apache/cloudstack
[Apache License version 2.0]: http://www.apache.org/licenses/LICENSE-2.0
[LICENSE]: https://github.com/scclouds/cloudstack-rpm-builder/blob/master/LICENSE
