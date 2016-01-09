## Contents
1. [Description](#description)
1. [Requirements](#requirements)
1. [Examples](#examples)
1. [Tags available](#tags-available)

## Description

CUDA images come in two flavors and are available through the [NVIDIA public hub repository](https://hub.docker.com/r/nvidia/cuda).

```runtime```: a lightweight image containing the bare minimum to deploy a pre-built application which uses CUDA.  
```devel```: extends the runtime image by adding the compiler toolchain, the debugging tools and the development files for the standard CUDA libraries. Use this image to compile a CUDA application from source.

## Requirements

Running a CUDA container requires a machine with at least one CUDA-capable GPU and a driver compatible with the CUDA toolkit version you are using.  
The machine running the CUDA container **only requires the NVIDIA driver**, the CUDA toolkit doesn't have to be installed.

NVIDIA drivers are **backward-compatible** with CUDA toolkits versions

CUDA toolkit version   | Minimum driver version  | Minimum GPU architecture
:---------------------:|:-----------------------:|:-------------------------:
  6.5                  | >= 340.29               | >= 2.0 (Fermi)
  7.0                  | >= 346.46               | >= 2.0 (Fermi)
  7.5                  | >= 352.39               | >= 2.0 (Fermi)

## Examples
Take a look at the [samples section](Testing-the-samples) to find examples of Dockerfiles compiling simple CUDA applications.

```sh
# Running an interactive CUDA session isolating the first GPU
nvidia-docker pull nvidia/cuda
NV_GPU=0 nvidia-docker run -ti --rm nvidia/cuda

# Querying the CUDA 7.5 compiler version
nvidia-docker pull nvidia/cuda:7.5-devel
nvidia-docker run --rm nvidia/cuda:7.5-devel nvcc --version
```

## Tags available

The following tags default to images based on [ubuntu:14.04](https://hub.docker.com/_/ubuntu/).   
Most of them are also available for other distributions. Selecting an alternative distribution is done by suffixing it to the tag (e.g `7.0-centos7` for CUDA 7.0 on CentOS 7).

Check the [tags reference](https://hub.docker.com/r/nvidia/cuda/tags/) for a given tag availability.

**CUDA only**
- [`7.5-runtime`, `runtime` (*ubuntu-14.04/cuda/7.5/runtime/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.5/runtime/Dockerfile)
- [`7.5-devel`, `7.5`, `devel`, `latest` (*ubuntu-14.04/cuda/7.5/devel/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.5/devel/Dockerfile)
- [`7.0-runtime` (*ubuntu-14.04/cuda/7.0/runtime/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.0/runtime/Dockerfile)
- [`7.0-devel`, `7.0` (*ubuntu-14.04/cuda/7.0/devel/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.0/devel/Dockerfile)
- [`6.5-runtime` (*ubuntu-14.04/cuda/6.5/runtime/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/6.5/runtime/Dockerfile)
- [`6.5-devel`, `6.5` (*ubuntu-14.04/cuda/6.5/devel/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/6.5/devel/Dockerfile)

**CUDA + cuDNN**
- [`7.5-cudnn4-runtime`, `cudnn-runtime` (*ubuntu-14.04/cuda/7.5/runtime/cudnn4/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.5/runtime/cudnn4/Dockerfile)
- [`7.5-cudnn4-devel`, `cudnn-devel`, `cudnn` (*ubuntu-14.04/cuda/7.0/devel/cudnn4/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.5/devel/cudnn4/Dockerfile)
- [`7.5-cudnn3-runtime` (*ubuntu-14.04/cuda/7.5/runtime/cudnn3/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.5/runtime/cudnn3/Dockerfile)
- [`7.5-cudnn3-devel` (*ubuntu-14.04/cuda/7.0/devel/cudnn3/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.5/devel/cudnn3/Dockerfile)
- [`7.0-cudnn4-runtime` (*ubuntu-14.04/cuda/7.0/runtime/cudnn4/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.0/runtime/cudnn4/Dockerfile)
- [`7.0-cudnn4-devel` (*ubuntu-14.04/cuda/7.0/devel/cudnn4/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.0/devel/cudnn4/Dockerfile)
- [`7.0-cudnn3-runtime` (*ubuntu-14.04/cuda/7.0/runtime/cudnn3/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.0/runtime/cudnn3/Dockerfile)
- [`7.0-cudnn3-devel` (*ubuntu-14.04/cuda/7.0/devel/cudnn3/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.0/devel/cudnn3/Dockerfile)
- [`7.0-cudnn2-runtime` (*ubuntu-14.04/cuda/7.0/runtime/cudnn2/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.0/runtime/cudnn2/Dockerfile)
- [`7.0-cudnn2-devel` (*ubuntu-14.04/cuda/7.0/devel/cudnn2/Dockerfile*)](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/7.0/devel/cudnn2/Dockerfile)