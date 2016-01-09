## Contents
1. [Description](#description)
1. [Building all tags](#building-all-tags)
1. [Building a specific tag](#building-a-specific-tag)
1. [Building custom distribution images](#building-custom-distribution-images)

## Description

If you don't want to rely on the [NVIDIA public hub repository](https://hub.docker.com/r/nvidia), you can always build images locally.  
Note that images can be built on any machine running Docker, **it doesn't require NVIDIA GPUs nor any driver installation**.
Images built locally will have the same names and tags as on the DockerHub but without the `nvidia/` prefix.

## Building all tags
You can build all existing tags of an image by using the top-level Makefile:
```sh
# Build all the CUDA images
make cuda
```
This command will build all the tags of the `cuda` image, as listed [here](CUDA#tags-available).

## Building a specific tag
To build a single tag of an image, use the Makefile for the image (instead of the top-level Makefile) and use the tag as the target:
```sh
# Build CUDA 7.0 runtime based on ubuntu 14.04
make -C ubuntu-14.04/cuda 7.0-runtime
```

## Building custom distribution images
By default, when building the images locally they are based on the official image `ubuntu:14.04`. Use the `OS` variable to build the images for other distributions:
```sh
# Build all the CUDA images for CentOS 6
make cuda OS=centos-6
```