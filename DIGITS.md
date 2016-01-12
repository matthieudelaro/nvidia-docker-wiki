## Contents
1. [Description](#description)
1. [Requirements](#requirements)
1. [Examples](#examples)
1. [Tags available](#tags-available)

## Description
Docker image based on [NVIDIA/caffe](https://github.com/NVIDIA/caffe), NVIDIA's fork of [BVLC/caffe](https://github.com/BVLC/caffe).

## Requirements
See the requirements for [CUDA](CUDA#requirements) since the DIGITS image is based on a cuDNN runtime image.

## Examples

```sh
# Run DIGITS on host port 8080
nvidia-docker run --name digits -d -p 8080:34448 nvidia/digits
```
If you want to use a dataset stored in a host directory, you will need to import it inside the container using a volume:
```sh
# Run DIGITS with the mnist dataset stored on the host under /opt/mnist
nvidia-docker run --name digits -d -p 8080:34448 -v /opt/mnist:/data/mnist nvidia/digits
```

If you want to share jobs between multiple DIGITS containers, you can use a named volume:
```sh
# Run DIGITS storing jobs in a host volume named digits-jobs
nvidia-docker run --name digits -d -p 8080:34448 -v digits-jobs:/usr/share/digits/digits/jobs nvidia/digits
```

## Tags available
- [`latest` (*ubuntu-14.04/digits/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/digits/Dockerfile)
