
# chrisbensch/docker-aptcacherng (all credit to sameersbn)

- [Introduction](#introduction)
  - [Contributing](#contributing)
  - [Issues](#issues)
- [Getting started](#getting-started)
  - [Installation](#installation)
  - [Quickstart](#quickstart)
  - [Command-line arguments](#command-line-arguments)
  - [Persistence](#persistence)
  - [Docker Compose](#docker-compose)
  - [Usage](#usage)
  - [Logs](#logs)
- [Maintenance](#maintenance)
  - [Cache expiry](#cache-expiry)
  - [Upgrading](#upgrading)
  - [Shell Access](#shell-access)

# Introduction

`Dockerfile` to create a [Docker](https://www.docker.com/) container image for [Apt-Cacher NG](https://www.unix-ag.uni-kl.de/~bloch/acng/).

Apt-Cacher NG is a caching proxy, specialized for package files from Linux distributors, primarily for [Debian](http://www.debian.org/) (and [Debian based](https://en.wikipedia.org/wiki/List_of_Linux_distributions#Debian-based)) distributions but not limited to those.

## Contributing

If you find this image useful here's how you can help:

- Send a pull request with your awesome features and bug fixes
- Support the development of this image with a [donation](http://www.damagehead.com/donate/)

# Getting started

## Installation


```bash
docker pull chrisbensch/docker-aptcacherng
```

Alternatively you can build the image yourself.

```bash
docker build -t chrisbensch/docker-aptcacherng github.com/chrisbensch/docker-aptcacherng
```

## Quickstart

Start Apt-Cacher NG using:

```bash
docker run --name apt-cacher-ng --init -d --restart=always \
  --publish 3142:3142 \
  --volume /srv/docker/apt-cacher-ng:/var/cache/apt-cacher-ng \
  chrisbensch/docker-aptcacherng
```

*Alternatively, you can use the sample [docker-compose.yml](docker-compose.yml) file to start the container using [Docker Compose](https://docs.docker.com/compose/)*

## Command-line arguments

You can customize the launch command of Apt-Cacher NG server by specifying arguments to `apt-cacher-ng` on the `docker run` command. For example the following command prints the help menu of `apt-cacher-ng` command:

```bash
docker run --name apt-cacher-ng --init -it --rm \
  --publish 3142:3142 \
  --volume /srv/docker/apt-cacher-ng:/var/cache/apt-cacher-ng \
  chrisbensch/docker-aptcacherng -h
```

## Persistence

For the cache to preserve its state across container shutdown and startup you should mount a volume at `/var/cache/apt-cacher-ng`.

> *The [Quickstart](#quickstart) command already mounts a volume for persistence.*

SELinux users should update the security context of the host mountpoint so that it plays nicely with Docker:

```bash
mkdir -p /srv/docker/apt-cacher-ng
chcon -Rt svirt_sandbox_file_t /srv/docker/apt-cacher-ng
```

## Docker Compose

To run Apt-Cacher NG with Docker Compose, create the following `docker-compose.yml` file

```yaml
---

services:
  apt-cacher-ng:
    image: chrisbensch/docker-aptcacherng
    container_name: apt-cacher-ng
    ports:
      - "3142:3142"
    volumes:
      - apt-cacher-ng:/var/cache/apt-cacher-ng
    restart: always

volumes:
  apt-cacher-ng:
```

The Apt-Cache NG service can then be started in the background with:

```bash
docker-compose up -d
```

## Usage

To start using Apt-Cacher NG on your Debian (and Debian based) host, create the configuration file `/etc/apt/apt.conf.d/01proxy` with the following content:

```config
Acquire::HTTP::Proxy "http://172.17.0.1:3142";
Acquire::HTTPS::Proxy "false";
```

Similarly, to use Apt-Cacher NG in you Docker containers add the following line to your `Dockerfile` before any `apt-get` commands.

```dockerfile
RUN echo 'Acquire::HTTP::Proxy "http://172.17.0.1:3142";' >> /etc/apt/apt.conf.d/01proxy \
 && echo 'Acquire::HTTPS::Proxy "false";' >> /etc/apt/apt.conf.d/01proxy
```

## Logs

To access the Apt-Cacher NG logs, located at `/var/log/apt-cacher-ng`, you can use `docker exec`. For example, if you want to tail the logs:

```bash
docker exec -it apt-cacher-ng tail -f /var/log/apt-cacher-ng/apt-cacher.log
```

# Maintenance

## Cache expiry

Using the [Command-line arguments](#command-line-arguments) feature, you can specify the `-e` argument to initiate Apt-Cacher NG's cache expiry maintenance task.

```bash
docker run --name apt-cacher-ng --init -it --rm \
  --publish 3142:3142 \
  --volume /srv/docker/apt-cacher-ng:/var/cache/apt-cacher-ng \
  chrisbensch/docker-aptcacherng -e
```

The same can also be achieved on a running instance by visiting the url http://localhost:3142/acng-report.html in the web browser and selecting the **Start Scan and/or Expiration** option.

## Upgrading

To upgrade to newer releases:

  1. Download the updated Docker image:

  ```bash
  docker pull chrisbensch/docker-aptcacherng
  ```

  2. Stop the currently running image:

  ```bash
  docker stop apt-cacher-ng
  ```

  3. Remove the stopped container

  ```bash
  docker rm -v apt-cacher-ng
  ```

  4. Start the updated image

  ```bash
  docker run --name apt-cacher-ng --init -d \
    [OPTIONS] \
    chrisbensch/docker-aptcacherng
  ```

## Shell Access

For debugging and maintenance purposes you may want access the containers shell. If you are using Docker version `1.3.0` or higher you can access a running containers shell by starting `bash` using `docker exec`:

```bash
docker exec -it apt-cacher-ng bash
```
