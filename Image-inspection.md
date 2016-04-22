## Contents
1. [Challenges](#challenges)
1. [nvidia-docker](#nvidia-docker)
1. [Alternatives](#alternatives)

## Challenge
Mounting user-level driver libraries and device files clobbers the environment of the container, it should be done only when the container is running a GPU application. The challenge here is to determine if a given image will be using the GPU or not.
We should also prevent launching containers based on a Docker image that is incompatible with the host NVIDIA driver version, you can find more details on this [wiki page](CUDA#requirements).    

## nvidia-docker
There is no generic solution for detecting if any image will be using GPU code, in `nvidia-docker` we assume that all images based on our `nvidia/cuda` images (available on the DockerHub) will be GPU applications and therefore they require the driver volume and the device files.  
More specifically, when `nvidia-docker run` is used, we inspect the image specified on the command-line. In this image, we lookup the presence and the value of the [label](https://docs.docker.com/engine/userguide/labels-custom-metadata/) `com.nvidia.volumes.needed`. The `nvidia/cuda` images we provide all include this label at the [beginning](https://github.com/NVIDIA/nvidia-docker/blob/64510511e3fd0d00168eb076623854b0fcf1507d/ubuntu-14.04/cuda/7.5/runtime/Dockerfile#L4). All the Dockerfiles that do a `FROM nvidia/cuda` will automatically inherit this metadata and thus will work seamlessly with `nvidia-docker`.

In order to detect when an image is not compatible with the host driver, we rely on a second piece of metadata, the [`com.nvidia.cuda.version` label](https://github.com/NVIDIA/nvidia-docker/blob/64510511e3fd0d00168eb076623854b0fcf1507d/ubuntu-14.04/cuda/7.5/runtime/Dockerfile#L15). This label is present in each of our base CUDA image, with the corresponding version number. This version is compared with the maximum CUDA version supported by the driver, `nvidia-docker` uses the CUDA API function `cudaDriverGetVersion` for this purpose. If the driver is too old for running this version of CUDA, an error is raised before starting the container:
```
$ nvidia-docker run --rm nvidia/cuda
nvidia-docker | 2016/04/21 21:41:35 Error: unsupported CUDA version: driver 7.0 < image 7.5
```

## Alternatives
In this case, `nvidia-docker` does not simply inject arguments to the `docker` command-line. As a result, it’s more complicated to reproduce this behavior. You would need to inspect the images upstream in your workflow or inside your container orchestration solution. Looking up a label inside an image is simple:
```
$ docker inspect --format='{{index .Config.Labels "com.nvidia.cuda.version"}}' nvidia/cuda
7.5
```
If you build your own custom CUDA images, we suggest you to reuse the same labels for compatibility reasons.
