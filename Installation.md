## Contents
1. [Prerequisites](#prerequisites)
1. [Installing from binaries](#installing-from-binaries)
1. [Building from sources](#building-from-sources)

## Prerequisites

The list of prerequisites for running `nvidia-docker` is described below.  
For information on how to install Docker for your Linux distribution, please refer to the [Docker documentation](https://docs.docker.com/engine/installation).

1. GNU/Linux x86_64
1. Docker >= 1.9
1. NVIDIA GPU with Architecture > Fermi (2.1)
1. [NVIDIA drivers](http://www.nvidia.com/object/unix.html)

> Your driver version might limit your CUDA capabilities (see [CUDA requirements](CUDA#requirements))

## Installing from binaries

## Building from sources

At the root directory of the repository issue `make` to build the binaries or alternatively `make install`.
The later will also take care of installing the binaries at the location set by the environment variable `PREFIX`(`/usr/bin` by default).  
`make clean` takes care of cleaning the build environment.