# Savings

Saving states of containers or images.

## Table of Contents

- [Committing changes](#committing-changes)
- [Saving images](#saving-images)
- [Exporting containers](#exporting-containers)
- [Summary](#summary)

### Committing changes

In this note, I need to \[re\]pull an Alpine Linux image. Then I would like to install [SQLite](https://www.sqlite.org/) and play with it. SQLite is a self-contained, serverless, zero-configuration SQL database engine good for small to medium-sized projects.

```shell
╰─➤  docker pull alpine:3.21
3.21: Pulling from library/alpine
897d797d2723: Pull complete
Digest: sha256:48b0309ca019d89d40f670aa1bc06e426dc0931948452e8491e3d65087abc07d
Status: Downloaded newer image for alpine:3.21
docker.io/library/alpine:3.21

╰─➤  docker run --name=my-alpine -it alpine:3.21

/ # apk add sqlite
(1/4) Installing ncurses-terminfo-base (6.5_p20241006-r3)
(2/4) Installing libncursesw (6.5_p20241006-r3)
(3/4) Installing readline (8.2.13-r0)
(4/4) Installing sqlite (3.48.0-r4)
Executing busybox-1.37.0-r14.trigger
OK: 9 MiB in 19 packages
/ # sqlite3 --version
3.48.0 2025-01-14 11:05:00 d2fe6b05f38d9d7cd78c5d252e99ac59f1aea071d669830c1ffe4e8966e84010 (64-bit)
/ # exit

╰─➤  docker container start -ai my-alpine
/ # sqlite3 --version
3.48.0 2025-01-14 11:05:00 d2fe6b05f38d9d7cd78c5d252e99ac59f1aea071d669830c1ffe4e8966e84010 (64-bit)
/ # exit
```

Cool, the progress is saved...but for how long? If the container is removed, all changes are lost because they are not written to the image which is good because images and containers are independent constructs and explicitness is crucial.

There is a way to save the state of a container by creating a new image from it using the [docker \[container\] commit](https://docs.docker.com/reference/cli/docker/commit/) command.
```shell
╰─➤  docker commit my-alpine sqlite-alpine
sha256:2ce85d28c03fa1d4b722c2a0071afb31f609d131c16b4320d5340a9d4fc324da

╰─➤  docker images
REPOSITORY                     TAG                         IMAGE ID       CREATED          SIZE
sqlite-alpine                  latest                      2ce85d28c03f   14 seconds ago   13MB
```

Removing the original "my-alpine" container and running an new one from the recently committed "sqlite-alpine" image:
```shell
╰─➤  docker container rm my-alpine
my-alpine

╰─➤  docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

╰─➤  docker run -it --name=my-sqlite sqlite-alpine
/ # sqlite3 --version
3.48.0 2025-01-14 11:05:00 d2fe6b05f38d9d7cd78c5d252e99ac59f1aea071d669830c1ffe4e8966e84010 (64-bit)
/ # exit

╰─➤  docker ps -a
CONTAINER ID   IMAGE           COMMAND     CREATED         STATUS                          PORTS     NAMES
231c217c4e64   sqlite-alpine   "/bin/sh"   2 minutes ago   Exited (0) About a minute ago             my-sqlite
```

I should have named it as "alpine-sqlite" because it is Alpine Linux with SQLite installed and the start command is `/bin/sh`, not `sqlite3`, for isntance. Still, this is a progress with proved commitment.

### Saving images

There is the "sqlite-alpine" image committed from the previously played "my-alpine" container. This image is stored on my machine and what if I need to share it? The problem is I have no reason to neithercreate a Docker Hub account nor any one for other registry services.

The solution I found is [docker \[image\] save](https://docs.docker.com/reference/cli/docker/image/save/) command that allows saving images to a tar archive.

```shell
╭─... ~/Projects/Learning-Docker/learnground/00_docker/01_savings  ‹savings*›
╰─➤  docker save sqlite-alpine:latest > alpine-sqlite.tar
```

I can even [gzip](https://manpages.debian.org/trixie/gzip/gzip.1.en.html) the recently created archive.

```shell
╭─... ~/Projects/Learning-Docker/learnground/00_docker/01_savings  ‹savings*›
╰─➤  gzip -k7 alpine-sqlite.tar
╭─... ~/Projects/Learning-Docker/learnground/00_docker/01_savings  ‹savings*›
╰─➤  ls
00_main.md  01_extra.md  alpine-sqlite.tar  alpine-sqlite.tar.gz
```

Ok, now deleting the original image and the related container - all is gone. But I have the archive and I can restore the image from it via the [docker \[image\] load](https://docs.docker.com/reference/cli/docker/image/load/) command.

```shell
╰─➤  docker load -i alpine-sqlite.tar.gz
f9486b7e0fc2: Loading layer [==================================================>]  5.237MB/5.237MB
Loaded image: sqlite-alpine:latest
╰─➤  docker image ls
REPOSITORY                     TAG                         IMAGE ID       CREATED        SIZE
sqlite-alpine                  latest                      2ce85d28c03f   6 days ago     13MB
...
```

Nice, time to run a container from the restored image and check the `sqlite3` version.

```shell
╰─➤  docker run -it --name=my-sqlite sqlite-alpine
/ # sqlite3 --version
3.48.0 2025-01-14 11:05:00 d2fe6b05f38d9d7cd78c5d252e99ac59f1aea071d669830c1ffe4e8966e84010 (64-bit)
/ # exit
```

Damn, it's awesome! The version and the hash sum are the same - this is the same state that I can upload and distribute with the archive and without any need in registry platforms. The "tar.gz" archive will be moved to the root-level [images](../../../images) directory and the plain ".tar" will be removed.

### Exporting containers

If we can archive the image state, can we also archive the container state? Yes, it is doable via the [docker \[container\] export](https://docs.docker.com/reference/cli/docker/container/export/) command.

Snippets go first:
```shell
╰─➤  docker run -it --name=my-alpine alpine:3.21
/ # apk update
fetch https://dl-cdn.alpinelinux.org/alpine/v3.21/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.21/community/x86_64/APKINDEX.tar.gz
v3.21.7-101-g4c306bf6940 [https://dl-cdn.alpinelinux.org/alpine/v3.21/main]
v3.21.7-99-gd268c03942f [https://dl-cdn.alpinelinux.org/alpine/v3.21/community]
OK: 25397 distinct packages available
/ # apk add wget curl
(1/11) Installing brotli-libs (1.1.0-r2)
(2/11) Installing c-ares (1.34.6-r0)
(3/11) Installing libunistring (1.2-r0)
(4/11) Installing libidn2 (2.3.7-r0)
(5/11) Installing nghttp2-libs (1.69.0-r0)
(6/11) Installing libpsl (0.21.5-r3)
(7/11) Installing zstd-libs (1.5.6-r2)
(8/11) Installing libcurl (8.14.1-r2)
(9/11) Installing curl (8.14.1-r2)
(10/11) Installing pcre2 (10.43-r0)
(11/11) Installing wget (1.25.0-r0)
Executing busybox-1.37.0-r14.trigger
OK: 13 MiB in 26 packages
/ # exit
╰─➤  docker ps -a
CONTAINER ID   IMAGE           COMMAND     CREATED              STATUS                          PORTS     NAMES
f3938b2b9c07   alpine:3.21     "/bin/sh"   About a minute ago   Exited (0) About a minute ago             my-alpine
9fe3e67f34a2   sqlite-alpine   "/bin/sh"   13 minutes ago       Exited (0) 13 minutes ago                 my-sqlite
```

Ok, I have a stopped (exited) container, I can (re)start it and check if [wget](https://www.gnu.org/software/wget/) and [curl](https://curl.se/) remain.

```shell
╰─➤  docker start -ai my-alpine
/ # wget --version
GNU Wget 1.25.0 built on linux-musl.
...
Originally written by Hrvoje Niksic <hniksic@xemacs.org>.
Please send bug reports and questions to <bug-wget@gnu.org>.

/ # curl --version
curl 8.14.1 (x86_64-alpine-linux-musl) ...
Release-Date: 2025-06-04
Protocols: dict file ftp ftps gopher gophers http https imap imaps ipfs ipns mqtt pop3 pop3s rtsp smb smbs smtp smtps telnet tftp ws wss
Features: alt-svc AsynchDNS brotli HSTS HTTP2 HTTPS-proxy IDN IPv6 Largefile libz NTLM PSL SSL threadsafe TLS-SRP UnixSockets zstd
/ # exit
```

Now, exporting the container (even if it is stopped) to the archive:
```shell
╰─➤  docker export my-alpine > alpine-curl-wget.tar
╰─➤  gzip -k7 alpine-curl-wget.tar
╰─➤  ls
00_main.md  01_extra.md  alpine-curl-wget.tar  alpine-curl-wget.tar.gz
```

There is a key difference between `docker save` and `docker export` commands: the former saves the image with all layers and metadata while the latter snapshots the container's filesystem at the export-time.

Before the go, pruning the alpine images and containers -> learning the "hard" way.
```shell
╰─➤  docker rm -f $(docker ps -aq)
f3938b2b9c07
9fe3e67f34a2
╰─➤  docker rmi sqlite-alpine
Untagged: sqlite-alpine:latest
Deleted: sha256:2ce85d28c03fa1d4b722c2a0071afb31f609d131c16b4320d5340a9d4fc324da
Deleted: sha256:3ba18b5c1cd4cec57bf9467fd10a6e7943ba9504de9444ecd061a154a015f654
```

Notes:

- `docker ps -aq` - list `-a`ll containers `-q`uietly -> only their IDs are displayed;
- `docker rm -f $(docker ps -aq)` - remove `-f`orcefully all containers listed by `docker ps -aq`.

This time I'll use the [docker \[image\] import](https://docs.docker.com/reference/cli/docker/image/import/) command that imports the conttents from a tarball to create a filesystem image. Yes, the image first because a container can be created from an image.

```shell
╰─➤  docker import -m "recreating alpine-curl-wget image" ./alpine-curl-wget.tar.gz curl-wget-alpine
sha256:333513041dbd84caac248f435e758dcfd8dce0890be1a42d40423ef70ebb11c2

╰─➤  docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

╰─➤  docker images
REPOSITORY                     TAG                         IMAGE ID       CREATED         SIZE
curl-wget-alpine               latest                      333513041dbd   9 seconds ago   16.4MB
```

The last "curl-wget-alpine" can be omitted, but if you do so, you'll get a `<none>:<none>` image without the repository and tag names. As you can see, specifying the repository name is enough because the tag wil lbe set to the default "latest".


```shell
╰─➤  docker run -it --name=my-wcurl-alpine curl-wget-alpine
docker: Error response from daemon: no command specified.
See 'docker run --help'.

╰─➤  docker run -it --name=my-wcurl-alpine curl-wget-alpine /bin/sh
/ # wget --version
GNU Wget 1.25.0 built on linux-musl.
...

/ # curl --version
curl 8.14.1 (x86_64-alpine-linux-musl) ...
/ # exit

╰─➤  docker ps -a
CONTAINER ID   IMAGE              COMMAND     CREATED              STATUS                      PORTS     NAMES
43c2b8cda771   curl-wget-alpine   "/bin/sh"   About a minute ago   Exited (0) 53 seconds ago             my-wcurl-alpine
```

I "don't know" about either the [docker \[image\] inspect](https://docs.docker.com/reference/cli/docker/image/inspect/) or the [docker \[container\] inspect](https://docs.docker.com/reference/cli/docker/container/inspect/) commands for now. But the last and the previous run demonstrated that the recreated image is not complete, I had to specify the `/bin/sh` command to run a container while for the original image it wasn't necessary. Intuitively the main difference between the `docker import` and `docker load` are similar to those for `docker export` and `docker save` correspondingly, but no rush for now until `docker inspect` to be introduced.

### Summary

- Wanna update container state and save it to the image -> `docker container commit`
- Willing to have the full archive of an image with layers, metadata and history? -> `docker image save`
- Care only about a lightweight snapshot of a container? -> `docker container export`
- Time to restore images? -> `docker image import` (URL, local tarball etc.) against `docker export` or `docker image load` from the `docker save`d tarball.
