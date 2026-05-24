# Introduction

A brief introduction to Docker, containers, images and some fundamental concepts.

## Table of Contents

- [History](#history)
- [Concepts](#concepts)
- [Basic container management](#basic-container-management)
- [Summary](#summary)

### History

Bad old days:

1. Installing an application A is impossible because it has dependencies incompatible with an application B. Installing the A breaks the B and vice versa.
2. On the dev server the application works great, yet it cannot be installed/deployed/run on the production server: OS (operating system), dependencies, configuration, whatever.
3. And other similar cases of incompatibilities and issues that make hard or even impossible running multiple applications on the same server.

The virtualisation technology was a game changer because it made possible to run multiple __isolated__ (!) applications on the same server. A virtual machine (VM) is a software emulation of a physical computer. Each VM has its own virtual (!) hardware: CPU, RAM, disks, network interfaces and so forth. Key features of VMs are: 1) isolation (from the host system and other applications); 2) hardware independence (the same VM can be run on different servers); 3) multiple OS support (the same VM can run different operating systems). However, VMs come with price vbecause running a VM requires emulating hardware and software resources -> this is a resource-intensive process with significant overhead.

The things would be easier if:

- there were no need in emulating hardware because a typical computer already has it;
- there were no point in emulating OS and other necessary software and just use what the computer already has;
- there were a way to create virtual environments or isolated sandboxes for applications.

These "dreams" came true with containers - they are like lightweight VMS because they do isolate/encapsulate applications without necessity of emulating hardware/software.

### Concepts

Quotes from the [Docker Overview](https://docs.docker.com/get-started/docker-overview/):

> An image is a read-only template with instructions for creating a Docker container. A container is a runnable instance of an image.

An image is like a blueprint or template for creating a container. It contains all the necessary instructions and dependencies for running an application.

### Basic container management

Let's play with a containerised application, so it be the [Alpine Linux lightweight distribution](https://www.alpinelinux.org/). Its Docker image is available on the [Docker Hub page](https://hub.docker.com/_/alpine/).

First, I want to check what images are already on my host machine via the `docker images` command.
```shell
╰─➤  docker images
REPOSITORY                     TAG       IMAGE ID       CREATED        SIZE
clamav/clamav                  latest    f0c30b4f8f54   2 months ago   238MB
clickhouse/clickhouse-server   latest    d0207e6d8732   7 months ago   799MB
```

- REPOSITORY - the name of the image repository (e.g. `alpine` or `clamav/clamav`)
- TAG - the version of the image
- IMAGE ID - the unique identifier of the image
- CREATED - when the image was created (not pulled from the Docker Hub)
- SIZE - the size of the image

First I need to [pull](https://docs.docker.com/reference/cli/docker/image/pull/) the Alpine Linux image from the Docker Hub, but not the latest one, so it be the version 3.21.

```shell
╰─➤  docker pull alpine:3.21
3.21: Pulling from library/alpine
897d797d2723: Pull complete 
Digest: sha256:48b0309ca019d89d40f670aa1bc06e426dc0931948452e8491e3d65087abc07d
Status: Downloaded newer image for alpine:3.21
docker.io/library/alpine:3.21
╰─➤  docker images
REPOSITORY                     TAG       IMAGE ID       CREATED        SIZE
alpine                         3.21      2607caa98058   4 weeks ago    7.83MB
clamav/clamav                  latest    f0c30b4f8f54   2 months ago   238MB
clickhouse/clickhouse-server   latest    d0207e6d8732   7 months ago   799MB
```

The `alpine:3.21` image was created 4 weeks ago, it is not the latest version (which is got by `docker [image] pull alpine` command when no tag is specified) and it is the smallest available image I have even though it's Linux OS.

A container - a runnable-instance of an image - can be just created (I don't need to run it now) via the [docker create](https://docs.docker.com/reference/cli/docker/container/create/) command. The `docker ps`, or [docker container ls](https://docs.docker.com/reference/cli/docker/container/ls/), command will list containers (by default, only running ones, that is why the `-a/--all` flag comes in handy).

```shell
╰─➤  docker create --memory=1G --cpus=1 --name=my-alpine -it alpine:3.21
98fc7bc071969733085c55c15e25958bfc4ddf44a9ae7e8bef5168fae7a464dd
─➤  docker ps --all
CONTAINER ID   IMAGE         COMMAND     CREATED          STATUS    PORTS     NAMES
98fc7bc07196   alpine:3.21   "/bin/sh"   24 seconds ago   Created             my-alpine
╰─➤  docker start my-alpine                                          
my-alpine
╰─➤  docker ps --all
CONTAINER ID   IMAGE         COMMAND     CREATED              STATUS          PORTS     NAMES
98fc7bc07196   alpine:3.21   "/bin/sh"   About a minute ago   Up 19 seconds             my-alpine
```

Actually. `docker ps` would do without `-a/--all` flag because, by default, it only shows running containers. Remarks on listed columns from the `docker ps` output:

- CONTAINER ID: the unique identifier of the container
- IMAGE: the image used to create the container
- COMMAND: the command that was executed when the container was created (here the shell)
- CREATED: the date and time the container was created
- STATUS: the current status of the container (up = running)
- PORTS: the ports that are exposed by the container (none)
- NAMES: the name of the container (if not specified, Docker will choose a random name)

And now, the Red-Letter moment, entering the container...technically, [exec](https://docs.docker.com/reference/cli/docker/container/exec/)uting the "/bin/sh" command in the running "my-alpine" container:

```shell
╰─➤  docker exec -it my-alpine /bin/sh
/ # ls
bin    etc    lib    mnt    proc   run    srv    tmp    var
dev    home   media  opt    root   sbin   sys    usr
/ #
/ # uname -a
Linux 98fc7bc07196 6.8.0-111-generic #111-Ubuntu SMP PREEMPT_DYNAMIC Sat Apr 11 23:16:02 UTC 2026 x86_64 Linux
/ # exit

╰─➤  docker ps
CONTAINER ID   IMAGE         COMMAND     CREATED          STATUS          PORTS     NAMES
98fc7bc07196   alpine:3.21   "/bin/sh"   18 minutes ago   Up 17 minutes             my-alpine
```

Yep "Linux 98fc7bc07196 ..." and the second part is the container's ID.

About the `-it` flags:

- `-i/--interactive` - keep STDIN (standard input stream) open even if not attached;
- `-t/--tty` - allocate a pseudo-TTY (TTY stands for "teletype (writer)", but referes to a terminal emulator).

Use cases:

1. No `-it` - can output, exits immediately after completing the command:

  ```shell
  ╰─➤  docker exec my-alpine /bin/sh -c "echo Hi"
  Hi
  ```

2. Only `--interactive` - STDIN is kept open without terminal features (TTY is not allocated):

  ```shell
  ╰─➤  docker exec -i my-alpine /bin/sh
  echo "Hi"
  Hi
  uname -a
  Linux 98fc7bc07196 6.8.0-111-generic #111-Ubuntu SMP PREEMPT_DYNAMIC Sat Apr 11 23:16:02 UTC 2026 x86_64 Linux
  exit
  ```

3. Only `--tty` - TTY is allocated, but STDIN is not kept open, so no actual interaction is in effect (you can play with <Tab> completions, in my case they didn't work):

  ```shell
  ╰─➤  docker exec -t my-alpine /bin/sh
  / # ^[[3;5R
  echo ""^[[DHi
  echo five
  exit
  oops
  ^Ccontext canceled
  ```

And only the `-it` combination gives fully interactive container terminal because STDIN are attached and TTY is allocated with terminal features.

Ok, time to [stop](https://docs.docker.com/reference/cli/docker/container/stop/) and rest. By default, Docker stops a container by sending first `SIGTERM` signal, and after a grace period, `SIGKILL`.

```shell
╰─➤  docker stop my-alpine
my-alpine
╰─➤  docker ps --all
CONTAINER ID   IMAGE         COMMAND     CREATED          STATUS                        PORTS     NAMES
98fc7bc07196   alpine:3.21   "/bin/sh"   45 minutes ago   Exited (137) 15 seconds ago             my-alpine
```

Not so fast, let's start the container again with either `docker start` or [docker container restart](https://docs.docker.com/reference/cli/docker/container/restart/) command, doesn't matter which one to use.

```shell
╰─➤  docker restart my-alpine
my-alpine
╰─➤  docker exec -it my-alpine /bin/sh
/ # apk update
fetch https://dl-cdn.alpinelinux.org/alpine/v3.21/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.21/community/x86_64/APKINDEX.tar.gz
v3.21.7-82-gdbf1ea75c8d [https://dl-cdn.alpinelinux.org/alpine/v3.21/main]
v3.21.7-80-g817c7780e5f [https://dl-cdn.alpinelinux.org/alpine/v3.21/community]
OK: 25397 distinct packages available
/ # apk add caddy
(1/2) Installing ca-certificates (20260413-r0)
(2/2) Installing caddy (2.8.4-r7)
Executing caddy-2.8.4-r7.pre-install
Executing busybox-1.37.0-r14.trigger
Executing ca-certificates-20260413-r0.trigger
OK: 52 MiB
/ # exit
```

If I restart the container again, the installed dependencies will remain.

```shell
╰─➤  docker exec -it my-alpine /bin/sh
/ # apk info --installed caddy  # installed before
caddy
/ # apk info --installed nginx  # were not installed
/ # ps -elf
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
   37 root      0:00 /bin/sh
   43 root      0:00 ps -elf
/ # exit
```

Restarting a running container is like `docker stop` followed by `docker start`. The main process (PID 1) is restarted, yet the container state is preserved.

If, for some reason, you don't want a container to consume CPU resources, you can [docker \[container\] pause](https://docs.docker.com/reference/cli/docker/container/pause/)
and [docker \[container\] unpause](https://docs.docker.com/reference/cli/docker/container/unpause/) it. Pausing a container is like freezing all processes inside a running container. The CPU for the paused processes is dropped to 0, but non-CPU resources (RAM, file descriptors, network connections, disks etc.) are not affected.

Now, time to remove the container with [docker \[container\] rm](https://docs.docker.com/reference/cli/docker/container/rm/) command. This time I don't need the previous `docker create --memory=1G --cpus=1 --name=my-alpine -it alpine:3.21`, I just wanna [docker \[container\] run](https://docs.docker.com/reference/cli/docker/container/run/) a container from the image.

```shell
╰─➤  docker stop my-alpine
my-alpine
╰─➤  docker container rm my-alpine
my-alpine
╰─➤  docker run --memory=1G --cpus=1 --name=my-alpine -it alpine:3.21

/ # apk update
fetch https://dl-cdn.alpinelinux.org/alpine/v3.21/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.21/community/x86_64/APKINDEX.tar.gz
v3.21.7-82-gdbf1ea75c8d [https://dl-cdn.alpinelinux.org/alpine/v3.21/main]
v3.21.7-80-g817c7780e5f [https://dl-cdn.alpinelinux.org/alpine/v3.21/community]
OK: 25397 distinct packages available
/ # apk info --installed caddy
/ # apk info --installed nginx
/ # apk add caddy
(1/2) Installing ca-certificates (20260413-r0)
(2/2) Installing caddy (2.8.4-r7)
Executing caddy-2.8.4-r7.pre-install
Executing busybox-1.37.0-r14.trigger
Executing ca-certificates-20260413-r0.trigger
OK: 52 MiB in 17 packages
/ # exit

╰─➤  docker ps --all
CONTAINER ID   IMAGE         COMMAND     CREATED              STATUS                      PORTS     NAMES
7543982c6beb   alpine:3.21   "/bin/sh"   About a minute ago   Exited (0) 48 seconds ago             my-alpine

╰─➤  docker exec -it my-alpine /bin/sh
/ # apk add caddy
OK: 52 MiB in 17 packages
/ # apk info --installed nginx
/ # apk info --installed caddy
caddy
/ # exit
```

Trying without the `-it` flags leads to an immediated quit because the primary process for an Alpine container is a shell (`/bin/sh`) and without STDIN and TTY it just exits. The fancy feature is that the container is created, therefore it can be \[re\]started with [Caddy](https://caddyserver.com/) dependency already in the container's system.

So, `docker run` is like `docker create` + `docker start` in one bottle. I thought that in that bottle comes `docker exec`, but it's not quite the same: `docker exec` attaches to an already running container and launches a command inside it without creating a new container, while `docker run` creates and starts a new container's from the image (if an image is not present, it is pulled first).

### Cleaning up

Let's list all containers in the system

```shell
─➤  docker ps -a
CONTAINER ID   IMAGE                                 COMMAND     CREATED        STATUS                    PORTS     NAMES
4a97d04fc82a   mcr.microsoft.com/powershell:alpine   "pwsh"      23 hours ago   Exited (0) 23 hours ago             gracious_lewin
7543982c6beb   alpine:3.21                           "/bin/sh"   45 hours ago   Exited (0) 44 hours ago             my-alpine
```

Their status is "Exited (0)" - execution suscessfully (status code 0) completed. The PowerShell container appeared throughout writing an [extra introductory note](./01_extra.md). Let's run it and try to remove via the [docker \[container\] rm (also remove)](https://docs.docker.com/engine/reference/commandline/container_rm/) command (also `docker container rm`, `docker container remove`, but not `docker remove`, see `docker rm --help`).

I run PowerShell in one terminal:
```shell
╰─➤  docker run -it mcr.microsoft.com/powershell:alpine pwsh

PowerShell 7.4.2
PS />
```

... and trying to remove the container in another one:
```shell
╰─➤  docker ps
CONTAINER ID   IMAGE                                 COMMAND   CREATED              STATUS          PORTS     NAMES
d693f1b99b68   mcr.microsoft.com/powershell:alpine   "pwsh"    About a minute ago   Up 59 seconds             thirsty_chatterjee

╰─➤  docker rm d693f1b99b68
Error response from daemon: cannot remove container "/thirsty_chatterjee": container is running: stop the container before removing or force remove
```

Let's remove it forcefully by sending `SIGKILL`:
```shell
╰─➤  docker rm -f d693f1b99b68
d693f1b99b68
```

And the PowerShell session is expectedly terminated.
```shell
╰─➤  docker run -it mcr.microsoft.com/powershell:alpine pwsh

PowerShell 7.4.2
PS /> % 
```

But what is that?!
```shell
╰─➤  docker ps -a
CONTAINER ID   IMAGE                                 COMMAND     CREATED        STATUS                    PORTS     NAMES
4a97d04fc82a   mcr.microsoft.com/powershell:alpine   "pwsh"      23 hours ago   Exited (0) 23 hours ago             gracious_lewin
7543982c6beb   alpine:3.21                           "/bin/sh"   45 hours ago   Exited (0) 44 hours ago             my-alpine
```

Nothing wrong because I ran a new container (named "thirsty_chatterjee") from the `mcr.microsoft.com/powershell:alpine` image, not the previously exited (named "gracious_lewin"). So...so long, gracious_lewin.

```shell
╰─➤  docker rm 4a97d04fc82a
4a97d04fc82a
```

The container is gone, but not the image and it's fair.

```shell
─➤  docker images
REPOSITORY                     TAG                         IMAGE ID       CREATED        SIZE
caddy                          2.11-alpine                 5190456911ce   9 days ago     62.9MB
ghcr.io/astral-sh/uv           0.11.13-python3.12-alpine   ad61364a0f5f   11 days ago    107MB
alpine                         3.21                        2607caa98058   5 weeks ago    7.83MB
clamav/clamav                  latest                      f0c30b4f8f54   2 months ago   238MB
clickhouse/clickhouse-server   latest                      d0207e6d8732   7 months ago   799MB
mcr.microsoft.com/powershell   alpine                      4591c431eb2f   2 years ago    222MB
```

But I don't need to remove it, I would like to remove the alpine:v3.21 (2607caa98058) image via the [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/) command (also `docker image rm`, `docker image remove`).

```shell
╰─➤  docker rmi 2607caa98058
Error response from daemon: conflict: unable to delete 2607caa98058 (must be forced) - image is being used by stopped container 7543982c6beb

╰─➤  docker rmi -f 2607caa98058
Untagged: alpine:3.21
Untagged: alpine@sha256:48b0309ca019d89d40f670aa1bc06e426dc0931948452e8491e3d65087abc07d
Deleted: sha256:2607caa9805847fac4de202017bb1b830deb09f4c07dc9964a0157abbc604577

╰─➤  docker images
REPOSITORY                     TAG                         IMAGE ID       CREATED        SIZE
caddy                          2.11-alpine                 5190456911ce   9 days ago     62.9MB
ghcr.io/astral-sh/uv           0.11.13-python3.12-alpine   ad61364a0f5f   11 days ago    107MB
clamav/clamav                  latest                      f0c30b4f8f54   2 months ago   238MB
clickhouse/clickhouse-server   latest                      d0207e6d8732   7 months ago   799MB
mcr.microsoft.com/powershell   alpine                      4591c431eb2f   2 years ago    222MB
```

Nice, but here is a peeky detail.

```shell
╰─➤  docker ps -a
CONTAINER ID   IMAGE          COMMAND     CREATED        STATUS                    PORTS     NAMES
7543982c6beb   2607caa98058   "/bin/sh"   45 hours ago   Exited (0) 44 hours ago             my-alpine
```

The image (template) is removed, but the container is an independent entity. Images are build-time constructs and containers are run-time ones. I can even start the container and it is not harmed by the image removal.

```shell
╰─➤  docker start -ai my-alpine
/ # echo Yeah
Yeah
/ # exit
```

Ok, sorry to se "my-alpine" fo, but it is what it is with `docker rm my-alpine`.

### Summary

Basics are over, key takeaways:

- A Docker image is a read‑only template with instructions for creating a Docker container. Think of it as a blueprint or snapshot that includes: the application code; runtime environment (e.g., Node.js, Python, Java); system tools and libraries; configuration files and settings; dependencies required to run the application.

- A Docker container is a runnable instance of an image. When you start (run) an image, it becomes a container. A container is the actual running application with its own isolated environment. Analogy: if an image is a program’s executable file (e.g., app.exe), then a container is that program running in memory (a process).

Overview of Docker commands:

- `docker pull alpine:3.21` - pull (download) an Alpine Linux image (!) from the Docker Hub (official docker container registry)
- `docker create [OPTIONS] IMAGE [COMMAND] [ARG...]` - create a container from the given image
- `docker start/stop/restart; pause/unpause` commands are used to control the lifecycle of a container. `docker restart` = `docker stop` + `docker start`
- `docker exec` - run a process in an already running container
- `docker run` = `docker create` + `docker start`
- get rid of conatainers with `docker rm` (also `docker container remove`) and say goodbye to images with `docker rmi` (or `docker image (rm | remove)`) commands (use `-f/--force` responsibly)

By the way, `docker start -ai my-alpine` is enough to start an interactive shell session in an Alpine Linux container, so no need to use the `docker exec` command.
