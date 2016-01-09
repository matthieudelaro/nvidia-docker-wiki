## Contents
1. [Benefits of GPU containerization](#benefits-of-gpu-containerization)
1. [Motivation](#motivation)
1. [Yet another Docker tool](#yet-another-docker-tool)

## Benefits of GPU containerization

Containerizing GPU applications provides several benefits, among them:

* Reproducible builds
* Ease of deployment
* Isolation of individual devices
* Run across heterogeneous driver/toolkit environments
* Requires only the NVIDIA driver to be installed
* Enables "fire and forget" GPU applications
* Facilitate collaboration

## Motivation
Docker containers are usually used to seamlessly deploy CPU-based applications on multiple machine. With this use case, Docker containers are both *hardware-agnostic* and *platform-agnostic*. This is obviously not the case when using NVIDIA GPUs since it is using specialized hardware and it requires the installation of the NVIDIA driver. As a result, Docker does not natively support NVIDIA GPUs with containers.

To solve this problem, one of the early solution that emerged was to fully reinstall the NVIDIA driver inside the container and then pass the character devices corresponding to the NVIDIA GPUs (e.g. `/dev/nvidia0`) when starting the container. However, this solution was brittle: the version of the host driver had to exactly match driver version installed in the container. The Docker images could not be shared and had to be built locally on each machine, defeating one of the main advantage of Docker.

To make the Docker images portable while still leveraging NVIDIA GPUs, the solution used by `nvidia-docker` is to make the images agnostic of the NVIDIA driver. The required character devices and driver files are mounted when starting the container on the target machine.

## Yet another Docker tool
Since this solution deals with nitty-gritty details and is quite different from the common Docker use cases, we provide two tools for convenience: an [alternative Docker CLI](Using nvidia-docker) and a [Docker plugin](Using nvidia-docker-plugin). While we understand that using supplementary command line tools can be frustrating, we tried to stay as close as possible from the Docker philosophy.

We hope that in the future Docker will enhance its plugin system to allow devices and parameters to be injected more easily in the command-line, rendering our tools obsolete.