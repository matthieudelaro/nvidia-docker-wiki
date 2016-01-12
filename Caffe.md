## Contents
1. [Description](#description)
1. [Requirements](#requirements)
1. [Examples](#examples)
1. [Tags available](#tags-available)

## Description
Docker image based on [NVIDIA/caffe](https://github.com/NVIDIA/caffe), NVIDIA's fork of [BVLC/caffe](https://github.com/BVLC/caffe).

## Requirements
See the requirements for [CUDA](CUDA#requirements) since the Caffe image is based on a cuDNN runtime image.

## Examples
This image can be used as a base image for other images like [[DIGITS]], or it can be used directly to train networks:
```sh
# Run a Caffe training job
nvidia-docker run --rm nvidia/caffe caffe train --solver <args>
```

## Tags available
- [`latest` (*ubuntu-14.04/caffe/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/caffe/Dockerfile)