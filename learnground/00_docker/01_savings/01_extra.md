# Savings (extra)

An extra note with supplementary jottings.

## Table of Contents

- [Own private registries](#own-private-registries)

### Own private registries

In the [00_main.md](./main.md) note I learnt how to create images from containers, how to archive images and containers and load them back into images. There were two reasons for that:

- trying the commands out;
- finding a way to distribute images/containers;
- feeling no need to sign up and in to the Docker Hub registry.

Then I thought about creating a private registry with minimum efforts. IN this note I share my findings.

```shell
╰─➤  docker run -d -p 5678:5000 --restart=always --name my-registry registry:latest

Unable to find image 'registry:latest' locally
latest: Pulling from library/registry
6a0ac1617861: Already exists
d5a24b5445d2: Pull complete
d1ddd084e87d: Pull complete
2c362d1d33f0: Pull complete
42b9845e5d5c: Pull complete
Digest: sha256:85347ed2ecde64161c7a4788a4d7d3dcc9d6f86f7be95834022e3c6a423a945a
Status: Downloaded newer image for registry:latest
1f50797a2fcb5fc090283974cd85428ded6b2a8a664b8f71f1cc7f49e281ec7e

╰─➤  docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                                         NAMES
1f50797a2fcb   registry:latest   "/entrypoint.sh /etc…"   18 seconds ago   Up 17 seconds   0.0.0.0:5678->5000/tcp, [::]:5678->5000/tcp   my-registry
```

Parameters:

- `-d`: run in detached mode ("daemonic");
- `-p 5678:5000`: map port 5678 on the host to port 5000 on the container;
- `--restart=always`: restart the container on failure;
- `--name my-registry`: name the container `my-registry`;
- `registry:latest`: use the `latest` tag of the `registry` image from Docker Hub.

Checking the repositories (none should be listed):
```shell
╰─➤  curl -X GET http://localhost:5678/v2/_catalog
{"repositories":[]}
```

Pulling the `hello-world` basic Docker image:
```shell
╰─➤  docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete
Digest: sha256:0e760fdfbc48ba8041e7c6db999bb40bfca508b4be580ac75d32c4e29d202ce1
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest
```

Before [docker \[image\] push](https://docs.docker.com/engine/reference/commandline/push/) the image I need to [docker \[image\] tag](https://docs.docker.com/engine/reference/commandline/tag/) it. Frome the docs:

> A Docker image reference consists of several components that describe where the image is stored and its identity. These components are `[HOST[:PORT]/]NAMESPACE/REPOSITORY[:TAG]`

- host -> the registry location where the image resides (`docker.io` (Docker Hub) is by default)
- port -> an optional port number (default is `:5000`)
- namespace/repository -> the namespace (optional) ussully represents a user or an organisation; and the repository (required) identifies the specific image.

By (re)tagging the existing "hello-world" image, I point the mew one to the local private registry and specify its namespace, repository, and tag there.

```shell
╰─➤  docker tag hello-world:latest localhost:5678/hello-world:latest

╰─➤  docker images
REPOSITORY                     TAG                         IMAGE ID       CREATED        SIZE
...
hello-world                    latest                      e2ac70e7319a   2 months ago   10.1kB
localhost:5678/hello-world     latest                      e2ac70e7319a   2 months ago   10.1kB
...
```

The retagged image is ready to be pushed to the local private registry.

```shell
╰─➤  docker push localhost:5678/hello-world:latest

The push refers to repository [localhost:5678/hello-world]
897b3f2a7c1b: Pushed
latest: digest: sha256:c766679d161d4ffe3dc4503b4c9f90b978f0d363fcedb02d1ae0cd271e645c0a size: 524
```

Drumroll, please...
```shell
╰─➤  curl -X GET http://localhost:5678/v2/_catalog
{"repositories":["hello-world"]}
```

Ta-dah, it's pushed! The final check is simple:

- remove the related target local image (I'll delete all of them)

```shell
╰─➤  docker rmi hello-world:latest localhost:5678/hello-world:latest
Untagged: hello-world:latest
Untagged: hello-world@sha256:0e760fdfbc48ba8041e7c6db999bb40bfca508b4be580ac75d32c4e29d202ce1

Untagged: localhost:5678/hello-world:latest
Untagged: localhost:5678/hello-world@sha256:c766679d161d4ffe3dc4503b4c9f90b978f0d363fcedb02d1ae0cd271e645c0a
Deleted: sha256:e2ac70e7319a02c5a477f5825259bd118b94e8b02c279c67afa63adab6d8685b
Deleted: sha256:897b3f2a7c1bc2f3d02432f7892fe31c6272c521ad4d70257df624504a3238b4
```

- pull it from the registry and run it locally.

```shell
╰─➤  docker pull localhost:5678/hello-world:latest

latest: Pulling from hello-world
4f55086f7dd0: Pull complete
Digest: sha256:c766679d161d4ffe3dc4503b4c9f90b978f0d363fcedb02d1ae0cd271e645c0a
Status: Downloaded newer image for localhost:5678/hello-world:latest
localhost:5678/hello-world:latest

╰─➤  docker images | grep hello-world
localhost:5678/hello-world     latest                      e2ac70e7319a   2 months ago   10.1kB

╰─➤  docker run localhost:5678/hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

Works for me.
