## Contents
1. [Prerequisites](#prerequisites)
1. [Installing from binaries](#installing-from-binaries)
1. [Building from sources](#building-from-sources)

## Prerequisites

The list of prerequisites for running `nvidia-docker` is described below.  
For information on how to install Docker for your Linux distribution, please refer to the [Docker documentation](https://docs.docker.com/engine/installation).

1. GNU/Linux x86_64 with kernel version > 3.10
1. Docker >= 1.9
1. NVIDIA GPU with Architecture > Fermi (2.1)
1. [NVIDIA drivers](http://www.nvidia.com/object/unix.html) >= 340.29

> Your driver version might limit your CUDA capabilities (see [CUDA requirements](CUDA#requirements))

## Installing from binaries

Binaries are available for download on the [release page](https://github.com/NVIDIA/nvidia-docker/releases).

Alternatively, deb packages are also available for Ubuntu based distributions: 
```
wget -P /tmp https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.0-beta.3/nvidia-docker_1.0.0.beta.3-1_amd64.deb
sudo dpkg -i /tmp/nvidia-docker_1.0.0.beta.3-1_amd64.deb && rm /tmp/nvidia-docker*.deb
``` 
The package installation will automatically setup the `nvidia-docker-plugin` and register it with the init system.

## Building from sources

At the root directory of the repository issue `make` to build the binaries or alternatively `make install`.
The later will also take care of installing the binaries at the location set by the environment variable `PREFIX`(`/usr/bin` by default).  
`make clean` takes care of cleaning the build environment.