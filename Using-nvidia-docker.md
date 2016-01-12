## Contents
1. [Description](#description)
1. [GPU isolation](#gpu-isolation)
1. [Running it locally](#running-it-locally)
1. [Running it remotely](#running-it-remotely)
1. [Known limitations](#known-limitations)

## Description

`nvidia-docker` is a drop-in replacement for the `docker` command line interface and it discovers host driver files, GPU devices, and sets up containers with NVIDIA GPUs for proper execution. For this purpose, a special Docker volume must be created before starting the containers. This volume can be created manually, or automatically by the [NVIDIA Docker plugin](Using nvidia-docker-plugin).

`nvidia-docker` also takes care of most of the boilerplate code related to [remote deployments](#running-it-remotely).

Internally, `nvidia-docker` calls `docker`. The command used by `nvidia-docker` can be overridden using the environment variable `NV_DOCKER`:
```sh
# Running nvidia-docker with a custom docker command
NV_DOCKER='sudo docker -D' nvidia-docker <docker-options> <docker-command> <docker-args>
```

## GPU isolation

GPUs are exported through a list of comma-separated IDs using the environment variable `NV_GPU`. The numbering is the same as reported by the `nvidia-docker-plugin` REST interface, `nvidia-smi`, or when running CUDA code with `CUDA_DEVICE_ORDER=PCI_BUS_ID`, it is however **different** from the default CUDA ordering. By default, all GPUs are exported.

```sh
# Running nvidia-docker isolating specific GPUs
NV_GPU='0,1' nvidia-docker <docker-options> <docker-command> <docker-args>
```

## Running it locally

#### With the NVIDIA Docker plugin
If the `nvidia-docker-plugin` is installed on your host and running locally, no additional step is needed. `nvidia-docker` will perform what is necessary by querying the plugin when containers using NVIDIA GPUs need to be launched.

#### Standalone version

If you want to rely solely on `nvidia-docker` to run your containers, you will initially need to setup the NVIDIA driver volumes on your local machine.
In order to do that, `nvidia-docker` provides an additional `setup` command to `docker volume`.
This is a one-time setup as long as the NVIDIA driver doesn't change. This operation requires root privileges.

```sh
# Setup the NVIDIA driver volume using the current driver
sudo nvidia-docker volume setup
```

In case you upgrade the NVIDIA drivers and want the change to be propagated to your containers, you will need to re-create the volume and re-run your existing containers (not images):

```sh
# Remove any previous setup
nvidia-docker volume rm `nvidia-docker volume ls -q | grep '^nvidia_.*'`
# Setup the NVIDIA driver volume using the updated driver
sudo nvidia-docker volume setup
```

> Executing `nvidia-docker run --rm` or `nvidia-docker rm -v` might remove some of the volumes required by `nvidia-docker`.  
> If that's the case, you will need to set up the volumes again.

## Running it remotely

Using `nvidia-docker` remotely requires the [NVIDIA Docker plugin](Using nvidia-docker-plugin) running on the remote host machine.  
The remote host target can be set using the environment variable `DOCKER_HOST` or `NV_HOST`.

The rules are as follows:
* If `NV_HOST` is set then it is used for contacting the plugin.
* If `NV_HOST` is **not** set but `DOCKER_HOST` is set then `NV_HOST` defaults to the `DOCKER_HOST` location  using the `http` protocol on port `3476` (more below)

The specification of `NV_HOST` is defined as:
```
[(http|ssh)://][<ssh-user>@][<host>][:<ssh-port>]:[<http-port>]
```

The `http` protocol requires the `nvidia-docker-plugin` to be listening on a reachable interface (by default `nvidia-docker-plugin` only listens on `localhost`). Opting for `ssh` however, only requires valid SSH credentials (either a password or a public key).

```sh
# Run CUDA on the remote host 10.0.0.1 using HTTP
DOCKER_HOST='10.0.0.1:' nvidia-docker run cuda

# Run CUDA on the remote host 10.0.0.1 using SSH
NV_HOST='ssh://10.0.0.1:' nvidia-docker -H 10.0.0.1: run cuda

# Run CUDA on the remote host 10.0.0.1 using SSH with custom user and ports
DOCKER_HOST='10.0.0.1:' NV_HOST='ssh://foo@10.0.0.1:22:80' nvidia-docker run cuda
```

## Known limitations

1. NVIDIA driver installation needs to exist on the same partition as the root of the Docker runtime.  
`nvidia-docker` internally needs to create hard links to some driver files. Because of this, you need the NVIDIA driver (usually found under `/usr`) to be on the same partition as the Docker root (`/var` by default).
Possible workarounds includes installing your NVIDIA driver at a different location (see ``--advanced-options`` of the installer) or changing your Docker root directory (see `docker daemon -g`)