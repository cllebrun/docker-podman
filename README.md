# Docker/Podman
This README explains the basics with demos.

Assignments (hands-on labs) for participants are in a different file:
[docker assignment](../docker-podman/docker-assignment.md) or [podman assignment](../docker-podman/podman-assignment.md)

## Standards

- Define image and container
- List images and containers
- Show running containers
- Pull and push images to quay.io
- Create a Dockerfile and build it
- Compare Docker and Podman
- Understand Environment Variables

## Lesson

Starting at [the documentation](https://www.docker.com/resources/what-container) we get an idea of what Docker is:

> A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.

The `container` concept is borrowed from shipping containers, which define a standard to ship goods globally. [Containerization](https://en.wikipedia.org/wiki/Containerization) dates back to early 18th Century coal mines. However, it wasn't until the 20th Century that _standardized_ containers became the norm. Docker does this in a modern way:

> Docker defines a standardized way to ship software.

Have you ever said or heard `It works on my machine!`? Docker lets you ship "your machine" to production by defining the runtime environment for the software and isolating that environment from everything else running on the physical hardware.

Similar to code, developers typically share Docker _images_, which are then run as _containers_. The location that we will store our Docker images is [Quay.io](https://quay.io/repository/).

There are a few key terms for working with Docker. Per the [Glossary](https://docs.docker.com/glossary):

- **Image** - The basis for containers. An image:
  - contains a union of layered filesystems stacked on top of each other
  - does not have state, and it never changes
  - that has no parent is a `base image`


- **Container** - a runtime instance of a `docker image`
  - Consists of
    - Docker image
    - execution environment
    - standard set of instructions


- **Registry** - a delivery and storage system for named Docker images.
  - Use `docker push` and `docker pull` to push and pull images from the registry
  - Companies usually have private docker registries
  - Similar to `maven`, `npm`, and `artifactory`
  - We will use [Quay.io](https://quay.io/) as our Docker registry


* **Tag** - a label applied to a Docker image in a repository.
  - How various images in a repository are distinguished from each other.
  - Can be almost anything you want
  - It is best to give it a meaningful, human-readable name that can be easily referenced later.

Seeing an example helps.

We begin with a classic [Hello, World!](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program) example.

Docker offers a lot, including the ability to download and run publicly available images. The most common spot for image hosting is [Dockerhub](https://hub.docker.com/).

We can easily see what Docker offers using `--help`:

```shell
  docker --help
```


We use `docker run` to start our first Docker container:

```shell
 docker run hello-world
```

`hello-world` is the name of the image on [Dockerhub](https://hub.docker.com/_/hello-world) that we want to run as a container.

It is worth reading the output carefully as it details the process Docker uses:

```shell
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

Following Docker's output, we try a slightly more ambitious example:

```shell
 docker run -it ubuntu bash
```

the flag `-i` indicates that we want an interactive session (i.e. keep the container running), and `-t` indicates [TTY](https://stackoverflow.com/questions/22272401/what-does-it-mean-to-attach-a-tty-std-in-out-to-dockers-or-lxc), `ubuntu` is the image name on Dockerhub and `bash` gives a Bash shell.

This image is not particular interesting though since it has only the base software of Ubuntu installed.

### Creating a Docker Image

What if we want to containerize something we created and publish it on Quay.io (or Dockerhub)? What would that require? A `Dockerfile`.

A `Dockerfile` is a ["text document that contains all the commands you would normally execute manually in order to build a Docker image."](https://docs.docker.com/glossary/). An example helps clarify this.

Create in a new folder in `~/workspace/` named `dockerfile-example`, cd into `dockerfile-example`. Next, `touch Dockerfile && code .`

We will populate the `Dockerfile` with:

```dockerfile
FROM alpine

CMD ["echo", "Hello, world! This is a simple Dockerfile using echo."]
```

In order to build the image we begin with(*):

```shell
 docker build --no-cache -t simple-dockerfile-example .
 docker run simple-dockerfile-example
A simple Dockerfile
```

While this is helpful for getting the basic workflow of `docker build` / `docker run` it is more instructive to see an application be built. For this example, we will adapt from the [Node with Docker Official Guide](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/).

To demonstrate how fast it is to go from project to containerized application we begin with a new Express app:

```shell
 npx express-generator app-to-containerize
 cd app-to-containerize && npm install
```

inside of `views/index.jade` we add a single line:

```
extends layout

block content
  h1= title
  p Welcome to #{title}
  p This is a Node Application that has been containerized with Docker!
```

next, we create a Dockerfile with `touch Dockerfile`. First, let's use Google or Dockerhub to see if there are any existing images we can use as a _base image_ for our build.

The [Official NodeJS](https://hub.docker.com/_/node) image should work:

```dockerfile
// Use node Docker image, version 16-alpine
FROM node:16-alpine

// From the documentation, "The WORKDIR instruction sets the working directory (folder) for any
// RUN, CMD, ENTRYPOINT, COPY and ADD instructions that follow it in the Dockerfile"
WORKDIR /usr/src/app

// COPY package.json and package-lock.json into root of WORKDIR
COPY package*.json ./

// Executes commands
RUN npm install

// Copies files from source to destination, in this case the root of the build context
// into the root of the WORKDIR
COPY . .

// Documents that the container process listens on port 3000
EXPOSE 3000

// Command to use for starting the application
CMD ["npm", "start"]
```

One important concept in the above example is the [build context](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#understand-build-context). In the simplest terms possible, the build context is the folder where we run `docker` commands. Our `Dockerfile` will typically be located at the same level of the file tree as the build context.

Since our build context, for this example, includes a folder of installed dependencies, `node_modules/`, we can use `.dockerignore` to ensure they are not copied into the image:

```docker
node_modules/
```

Next we build the image and start a container with this image:

```shell
 docker build --no-cache -t express-example-app .
 docker run --rm  -p 127.0.0.1:8080:3000 express-example-app
```
The `-p 127.0.0.1:8080:3000` indicates a port mapping from `localhost` to container host, and if we visit `http://localhost:8080` in a browser, then we see the sample express application.

A few useful commands for examining the state of the Docker container:

- `docker ps -a` to list all docker containers
- `docker images` to list all images installed
- `docker logs <container-id>` - shows the logs for a given container, with `-f` flag (similar to `tail -f`) waits for logs to be appended to
- `docker rm $(docker ps -qa)` to remove all stopped containers (compare output of `docker ps -qa` and `docker ps -a`)

### Podman
For a long time, Docker has been the most popular container management engine on the market. However, as containerization became the norm in the DevOps world, competitors like Podman emerged.
Podman has functionally the same as docker, although generally runs in a more secure way and is also open source.

Podman does not have a desktop GUI component by default, although there are some other 3rd party desktop GUIs available, or in development, that work with Podman. Podman is not a replacement for docker desktop as a whole, but is a replacement for the underlying docker cli and runtime. Podman is driven by command line.

The official podman installation instructions from the containers organisation are to be found https://podman.io/getting-started/installation [External Site]. Podman is part of RedHat's container offerings, but is available on other platforms.

We have given the most common method of downloading podman in each section below. However, there are other methods to download and install podman, such as directly from github (e.g https://github.com/containers/podman/releases/latest [External Site]). When downloading from any source, you should check the version downloaded is the correct one and that the checksums (normally in the shasums file in the github release area for podman) match what you have, before use.

There are more hints and community help in the #guild-podman channel

Containers simplify the production, distribution, discoverability, and usage of applications with all of their dependencies and default configuration files. Users test drive or deploy a new application with one or two commands instead of following pages of installation instructions. Here’s how to find your first Container Image:
```shell
podman search busybox
```

The previous command returned a list of publicly available container images on DockerHub. These container images are easy to consume, but of differing levels of quality and maintenance. Let’s use the first one listed because it seems to be well maintained.

To run the busybox container image, it’s just a single command:
```shell
podman run -it docker.io/library/busybox
```

Sometimes we can find a publicly available container image for the exact workload we’re looking for and it will already be packaged exactly how we want. But, more often than not, there’s something that we want to add, remove, or customize. It could be as simple as a configuration setting for security or performance, or as complex as adding a complex workload. Either way, containers make it fairly easy to make the changes we need.

Container Images aren’t actually images, they’re repositories often made up of multiple layers. These layers can easily be added, saved, and shared with others by using a Containerfile (Dockerfile). This single file often contains all the instructions needed to build a new container image and can easily be shared with others publicly using tools like GitHub.

Here’s an example of how to build a Nginx web server on top of a Debian base image using the Dockerfile maintained by Nginx and published in GitHub:
```shell
podman build -t nginx https://git.io/Jf8ol
```

You can see that image now exists on your machine with the following command:

```shell
podman image ls
```
Once, the image build completes, it’s easy to run the new image from our local cache:
```shell
podman run -d -p 8080:80 nginx
curl localhost:8080
```
Stop the container:

```shell
podman ps
podman stop [containerid]
```
Building new images is great, but sharing our work with others let’s them review our work, critique how we built them, and offer improved versions. Our newly built Nginx image could be published at quay.io or docker.io to share it with the world. Everything needed to run the Nginx application is provided in the container image. Others could easily pull it down and use it, or make improvements to it.

### Podman vs Docker

Podman is a daemonless, rootless container engine developed by RedHat, designed as an alternative to Docker. The modular design allows Podman to use individual system components only when needed. Its rootless approach to container management allows containers to be run by non-root users.

######  Architecture
Docker uses a daemon, an ongoing program running in the background, to create images and run containers. Podman has a daemon-less architecture which means it can run containers under the user starting the container. Docker has a client-server logic mediated by a daemon; Podman does not need the mediator.

######  Ease of Use
Docker features a comprehensive set of straightforward and intuitive commands. Using Docker, developers can easily create, deploy, and manage containerized applications.

Podman was built to seamlessly replace Docker in a software development workflow, so its commands are mostly the same as Docker's. For example, the docker pull command becomes podman pull.
Aside from Podman inheriting the ease-of-use of Docker, the similarity between the two tools also means that the migration from Docker to Podman requires little effort.
######  Root Privileges
Podman, since it doesn't have a daemon to manage its activity, also dispenses root privileges for its containers. Docker recently added rootless mode to its daemon configuration, but Podman used this approach first and promoted it as a fundamental feature. And this is because of the next point.
######  Security
Is Podman safer than Docker? Podman allows for non-root privileges for containers.Rootless containers are considered safer than containers with root privileges. In Docker, daemons have root privileges, making them the preferred gateway for attackers. Containers in Podman do not have root access by default, adding a natural barrier between root and rootless levels, improving security. Still, Podman can run both root and rootless containers.

######  Building images
As a self-sufficient tool, Docker can build container images on its own. Podman requires the assistance of another tool called Buildah(another open-source tool), which expresses its specialized nature: it is made for running but not building containers on its own.
When podman build is executed, the buildah bud (build-using-dockerfile) command is called to emulate the docker build command.
######  Running Containers
When Docker receives the docker run command, it performs multiple actions:

    1. Checks if the image the user-specified exists locally. If not, it pulls the image from the configured registries.
    2. Creates a writeable container layer on top of the image.
    3. Starts the container.

Running containers with Podman is performed using the podman run command, which functions the same way as docker run. One of the main benefits of Podman compared to Docker is that Podman fully integrates with systemd by default. This enables Podman to run systemd within the container out of the box.
######  Docker Swarm and Docker Compose
Docker Swarm is a container orchestration platform used to manage Docker containers. It enables developers to run a cluster of Docker nodes and deploy a scalable application without other dependencies required.

Podman does not support Docker Swarm. However, Podman users can use tools such as Nomad, which comes with a Podman driver.

To summerize:

||Docker|Podman|
|--------|--------|--------|
|    Daemon    |    Uses the Docker daemon    |    Daemonless architecture   |
|    Root    |   Runs containers as root only   |    Runs containers as root and as non-root    |
|    Images    |    Can build container images    |    Uses Buildah to build container images    |
|    Monolithic platform    |    Yes    |    No    |
|    Docker-swarm  |    Supported     |    Not supported   |
|    Docker-compose    |    Supported    |    Supported    |
|    Runs natively on   |    Linux, macOS, Windows    |    Linux, macOS, Windows (with WSL)   |		






### Environment Variables

_Question:_ If the Docker container is an independent runtime, then how do we inject environment-based values?

One principle of [12-factor application](https://12factor.net/) development is [config](https://12factor.net/config) or _store config in the environment_. A real world example of why this is important is that we do not want to use something like a database between development, test, and production. _Why?_ Because it is easy to make mistakes with no environment isolation, especially if the configuration is hard coded into the codebase and conditionally applied.

A small example helps demonstrate this concept.

1. `cd ~/workspace`
1. `mkdir docker-lister && cd docker-lister`.
1. Create a file named `lister.sh` that contains the following Bash script that shows the current folder, contents of the folder, and value of environment variables:

```shell
#! /usr/bin/env sh

echo The current working directory is:
echo
pwd
echo
echo ------------------------------------

echo Here are the contents of the working directory:
echo
ls
echo
echo ------------------------------------

echo Here are the environment variables:
echo
env
echo
echo ------------------------------------
```

1. To test this locally, make `lister.sh` executable with `chmod +x ./lister.sh`
1. Run `./lister.sh` and review the output
1. Containerize the script using Docker, first `touch Dockerfile`
1. Our Dockerfile will once again use `alpine` for simplicity:

```Dockerfile
FROM alpine

RUN mkdir app
WORKDIR app

COPY . .

CMD ["./lister.sh"]
```

1. `docker build -t ibm/lister .` (**Warning:** Do not use your root folder, `/`, as the PATH as it causes the build to transfer the entire contents of your hard drive to the Docker daemon.)
1. `docker run ibm/lister` - compare this output to `./lister.sh`'s output
1. `docker run -e API_URL=https://thecatapi.com/ ibm/lister` - compare this output to the others

The `-e` flag allowed us to inject the pair `API_URL` / `https://thecatapi.com/` into the container's environment.

`lister.sh` demonstrates that **the Docker container is a self-contained environment**. In each instance we see the working directory (folder), its contents, and the value of environment variables depends on where `lister.sh` is running.

(*) Please note that the --no-cache option when using docker build is optional. Docker images are multi-layered files containing multiple image layers on top of each other. Each instruction mentioned inside a Dockerfile is responsible for creating a new layer. A layer only consists of the differences between the previous layer and the current layer. If you have previously built the same image, the daemon will look for a cache containing the same image layer. If a subsequent cache is found, it will simply use this cache and not build a new layer.

However, there might be situations where you want to force a clean build of the image even if the build cache of subsequent layers exists. When you use the Docker build command to build a Docker image, you can simply use the --no-cache option which will allow you to instruct daemon to not look for already existing image layers and simply force clean build of an image.

## Assignment

Please complete the assignment [HERE](assignment.md)
## Glossary

> Build once, run everywhere.

| Term      | Description                                                                                                                                                                                              |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Container | Runtime instance of a docker image. Standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another.     |
| Docker    | Defines a standardized way to ship software (using containers).                                                                                                                                          |
| Podman       | Podman has functionally the same as docker, although generally runs in a more secure way and is also open source (developed by RedHat)                                                                                                                   |
| Image     | Union of layered filesystems. Lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings. |
| Registry  | Delivery and storage system for named Docker images.                                                                                                                                                     |
| Tag       | Label applied to a Docker image in a repository.                                                                                                                                                         |
