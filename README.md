# Docker Volume Driver For Linode

[![GoDoc](https://godoc.org/github.com/linode/docker-volume-linode?status.svg)](https://godoc.org/github.com/linode/docker-volume-linode)
[![Go Report Card](https://goreportcard.com/badge/github.com/linode/docker-volume-linode)](https://goreportcard.com/report/github.com/linode/docker-volume-linode)
[![Build Status](https://travis-ci.org/linode/docker-volume-linode.svg?branch=master)](https://travis-ci.org/linode/docker-volume-linode)

This [volume plugin](https://docs.docker.com/engine/extend/plugins_volume/) gives Docker the ability to manage [Linode Block Storage](https://www.linode.com/blockstorage) volumes using `docker volume` commands. 
[Good use cases for volumes](https://docs.docker.com/storage/#good-use-cases-for-volumes) include off-node storage to avoid size constraints or moving a container and the related volume between nodes in a [Swarm](https://github.com/linode/docker-machine-driver-linode#provisioning-docker-swarm).

## Requirements

- Linux (tested on Ubuntu 18.04, should work with other versions and distros)
- Docker (tested on version 17, should work with other versions)

## Installation

### Install and Configure (one step)

```sh
docker plugin install --alias linode linode/docker-volume-linode linode-token=<linode token> linode-region=<linode region> linode-label=<linode label>
```

### Install and Configure (separate steps)

```sh
docker plugin install --alias linode linode/docker-volume-linode
docker plugin disable linode
docker plugin set linode linode-token=<linode token>
docker plugin set linode linode-region=<linode region>
docker plugin set linode linode-label=<linode label>
docker plugin enable linode
```

- \<linode token\>: Token must be generated usigng Linode Control Panel https://login.linode.com.  The generated API Token must have Read/Write permission for Volumes and Linodes.
- \<linode regions\>: us-east, us-central, us-southeast, us-west, eu-west, eu-central, ap-south, ap-northeast, ap-northeast-1a
- \<linode label\>: The label given to the host Linode Control Panel.
- For a complete list of regions:  https://api.linode.com/v4/regions
- For all options see "Driver Options" section


### Docker Swarm

- The plugin must be installed in all nodes.
- Volumes can be mounted to one container at the time due to volumes been only attachable to one node at the time.


## Usage

All examples assume the driver has been aliased to `linode`.

### Create Volume

Linode Block Storage volumes can be created and managed using the [docker volume create](https://docs.docker.com/engine/reference/commandline/volume_create/) command.

```sh
$ docker volume create -d linode my-test-volume
my-test-volume
```

#### Create Options

This driver offers `size` as [driver specific option](https://docs.docker.com/engine/reference/commandline/volume_create/#driver-specific-options).  The `size` option specificies the size (in GB) of the volume to be created.  Volumes must be atleast 10GB in size, so the default is 10GB.

```sh
$ docker volume create -o size=50 -d linode my-test-volume-50
my-test-volume-50
```

### List Volumes

```sh
$ docker volume ls
DRIVER              VOLUME NAME
linode:latest       my-test-volume
linode:latest       my-test-volume-50
```

### Use Volume

```sh
$ docker run --rm -it -v my-test-volume:/usr/local/apache2/htdocs/ httpd
...
```

### Remove Volumes

```sh
$ docker volume rm my-test-volume
my-test-volume

$ docker volume rm my-test-volume-50
my-test-volume-50
```

### Driver Options

| Option Name | Description |
| --- | --- |
| linode-token | **Required** The Linode APIv4 [Personal Access Token](https://cloud.linode.com/profile/tokens)
| linode-label | **Required** The Linode Label to attach block storage volumes to (defaults to the system hostname) |
| linode-region | The Linode region to create volumes in (inferred if using linode-label, defaults to us-west) |
| socket-file | Sets the socket file/address (defaults to /run/docker/plugins/linode.sock) |
| socket-gid | Sets the socket GID (defaults to 0) |
| mount-root | Sets the root directory for volume mounts (default /mnt) |
| log-level | Sets log level to debug,info,warn,error (defaults to info) |



Options can be set once for all future uses with [`docker plugin set`](https://docs.docker.com/engine/reference/commandline/plugin_set/#extended-description).

## Manual Installation

### Requirements

- Install Golang: <https://golang.org/>
- Get code and Compile: `go get -u github.com/linode/docker-volume-linode`

### Run the driver

```sh
docker-volume-linode --linode-token=<token from linode console> --linode-region=<linode region> --linode-label=<linode label>
```

### Debugging

#### Enable Debug Level on plugin

The driver name when running manually is the same name as the socket file.

```sh
docker plugin set docker-volume-linode LOG_LEVEL=debug
```

#### Enable Debug Level in manual installation

```sh
docker-volume-linode --linode-token=<...> --linode-region=<...> --linode-label=<...> --log-level=debug
```

## Tested On

```text
Ubuntu 18.04 LTS
```

```text
Tested With:
Client:
 Version:       17.12.1-ce
 API version:   1.35
 Go version:    go1.10.1
 Git commit:    7390fc6
 Built: Wed Apr 18 01:23:11 2018
 OS/Arch:       linux/amd64

Server:
 Engine:
  Version:      17.12.1-ce
  API version:  1.35 (minimum version 1.12)
  Go version:   go1.10.1
  Git commit:   7390fc6
  Built:        Wed Feb 28 17:46:05 2018
  OS/Arch:      linux/amd64
  Experimental: false
```
