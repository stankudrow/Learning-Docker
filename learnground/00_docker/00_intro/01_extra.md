# Introduction (extra)

An extra note with supplementary jottings.

## Table of Contents

- [Container Registries](#container-registries)
- [Windows containers on Linux](#windows-containers-on-linux)
- [Summary](#summary)

### Container Registries

Let's pull the [Caddy](https://caddyserver.com/) server official [image](https://hub.docker.com/_/caddy) base on Alpine Linux.

```shell
╰─➤  docker pull caddy:2.11-alpine
2.11-alpine: Pulling from library/caddy
6a0ac1617861: Pull complete
5920de1d55d9: Pull complete
057c497bda06: Pull complete
3b55c68e4a0f: Pull complete
4f4fb700ef54: Pull complete
Digest: sha256:86deaf5e3d3408a6ccec08fbb79989783dd26e206ae10bcf78a801dc8c9ab794
Status: Downloaded newer image for caddy:2.11-alpine
docker.io/library/caddy:2.11-alpine
```

Caddy was pulled from the Docker Hub registry via the [docker \[image\] pull](https://docs.docker.com/engine/reference/commandline/pull/) command. **Attention to the digest hash (will be covered soon)**. The "caddy:2.11-alpine" consists of several parts:

- registry domain: `docker.io` (the last line of the output)
- namespace: `library` (the second line of the output)
- repository: `caddy`
- tag: `2.11-alpine`

A tag is the version of a concrete image in a repository. A repository is a collection of images with the same name but different tags. A namespace helps to logically grouprepositories within the registry. A registry domain is the server (registry) address. So, the full path to the image is "docker.io/library/caddy:2.11-alpine":

```shell
╰─➤  docker pull docker.io/library/caddy:2.11-alpine
2.11-alpine: Pulling from library/caddy
Digest: sha256:86deaf5e3d3408a6ccec08fbb79989783dd26e206ae10bcf78a801dc8c9ab794
Status: Image is up to date for caddy:2.11-alpine
docker.io/library/caddy:2.11-alpine
```

The digest hash is a unique identifier for the image, used to verify its integrity. I can be sure that I deal with the same image by checking the digest hash sums. If a single bit of an image is changed, its digest will also change.

The full usage "signature" of the `docker pull` command is `docker image pull [OPTIONS] NAME[:TAG|@DIGEST]` (the word image is optional).

```shell
╰─➤  docker image pull --help

Usage:  docker image pull [OPTIONS] NAME[:TAG|@DIGEST]

Download an image from a registry

Aliases:
  docker image pull, docker pull

Options:
  -a, --all-tags                Download all tagged images in the
                                repository
      --disable-content-trust   Skip image verification (default true)
      --platform string         Set platform if server is
                                multi-platform capable
  -q, --quiet                   Suppress verbose output
```

Let's pull the image again and this time by its digest hash:

```shell
╰─➤  docker image pull caddy@sha256:86deaf5e3d3408a6ccec08fbb79989783dd26e206ae10bcf78a801dc8c9ab794
docker.io/library/caddy@sha256:86deaf5e3d3408a6ccec08fbb79989783dd26e206ae10bcf78a801dc8c9ab794: Pulling from library/caddy
Digest: sha256:86deaf5e3d3408a6ccec08fbb79989783dd26e206ae10bcf78a801dc8c9ab794
Status: Image is up to date for caddy@sha256:86deaf5e3d3408a6ccec08fbb79989783dd26e206ae10bcf78a801dc8c9ab794
docker.io/library/caddy@sha256:86deaf5e3d3408a6ccec08fbb79989783dd26e206ae10bcf78a801dc8c9ab794
```

Awesome! Now, let's download the image if the [uv](https://docs.astral.sh/uv/) project manager. The [GitHub Packages page](https://github.com/orgs/astral-sh/packages?repo_name=uv) lists available versions and I am interested in "0.11.13-python3.12-alpine" (checkable [here](https://github.com/astral-sh/uv/pkgs/container/uv/versions?filters%5Bversion_type%5D=tagged)). The thing is that I need to pull it from the GitHub domain, so it should be specified for the Docker client is opinionated and defaults to Docker Hub.

```shell
╰─➤  docker pull ghcr.io/astral-sh/uv:0.11.13-python3.12-alpine
0.11.13-python3.12-alpine: Pulling from astral-sh/uv
6a0ac1617861: Already exists
254ac41e2afd: Pull complete
fd21a26fb55d: Pull complete
3a4f2e6e1560: Pull complete
7016470ceecf: Pull complete
Digest: sha256:afbc53fcac79bd7dd4c0d94a427bbf245cfe1a3283a2024a52db0ca266de77ed
Status: Downloaded newer image for ghcr.io/astral-sh/uv:0.11.13-python3.12-alpine
ghcr.io/astral-sh/uv:0.11.13-python3.12-alpine
```

To pull the image, I had to specify the full path to it with:

- domain: `ghcr.io` (GitHub Container Registry)
- namespace: `astral-sh`
- repository: `uv`
- tag: `0.11.13-python3.12-alpine`

Well, that is all for now. To summarise, a container registry is a service (or system) where images are pushed to and pull from.

### Windows containers on Linux

I work on Linux and I wonder if pulling and working with Windows containers is possible? In lieu of theorising, let's try it out:

```shell
╰─➤  docker pull mcr.microsoft.com/powershell:alpine
alpine: Pulling from powershell
3c854c8cbf46: Pull complete
d55f89a05da7: Pull complete
930ddab1c368: Pull complete
Digest: sha256:3346adafa19803def81b7ae5421cbfcc7ba077a14cbcd6b5447ff3751d644e13
Status: Downloaded newer image for mcr.microsoft.com/powershell:alpine
mcr.microsoft.com/powershell:alpine
```

And since now I can learn PowerShell on Linux! For example:

```shell
╰─➤  docker run -it mcr.microsoft.com/powershell:alpine pwsh

PowerShell 7.4.2
PS /> $PSVersionTable.PSVersion

Major  Minor  Patch  PreReleaseLabel BuildLabel
-----  -----  -----  --------------- ----------
7      4      2                      

PS /> Get-Date

Thursday, May 21, 2026 8:34:33 PM

PS /> exit
```

Damn, I was too quick to delete those PowerShell scripting books :)

### Containers up and down

I mentioned `docker [container] [un]pause` command in the [00_main.md](./00_main.md) note, but skipped it there. Here I wanna cover this subject. So, running a PowerShell container:

A terminal A:
```shell
╰─➤  docker run -it --name=psh mcr.microsoft.com/powershell:alpine pwsh

PowerShell 7.4.2
PS />
```

Then, pausing the container in another terminal B:
```shell
╰─➤  docker container pause psh
psh
╰─➤  docker ps
CONTAINER ID   IMAGE                                 COMMAND   CREATED         STATUS                  PORTS     NAMES
1f1357540a34   mcr.microsoft.com/powershell:alpine   "pwsh"    3 minutes ago   Up 3 minutes (Paused)             psh

╰─➤  docker ps --filter "status=paused"
CONTAINER ID   IMAGE                                 COMMAND   CREATED         STATUS                  PORTS     NAMES
1f1357540a34   mcr.microsoft.com/powershell:alpine   "pwsh"    3 minutes ago   Up 3 minutes (Paused)             psh
```

Typing in the PowerShell container associated TTY (the terminal A) produces no response -> all processes are frozed. When I run `docker [container] unpause psh` in another terminal, the container resumes and all inputs entered while the container was paused get executed immediately.

```shell
PowerShell 7.4.2
PS />
PS />
PS /> hello
hello: The term 'hello' is not recognized as a name of a cmdlet, function, script file, or executable program.
Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
PS /> exit
```

Let's run an alpine container, then pause it and try to `docker container exec` some command:

```shell
╰─➤  docker exec -it test-alpine /bin/sh
Error response from daemon: Container test-alpine is paused, unpause the container before exec
```

That was expected, I cannot run a process because the container is freezed, the CPU usage drops to 0%.

### Summary

A registry is a service (or system) where images are stored. Images are stored in repositories, versioned with tags and are identified by digests (hash sums). Repositories itself can be grouped with namespaces.

Also, when `docker run` an image that does not exist locally, Docker will pull it from the registry (DockerHub by default).
