## Contents
1. [Description](#description)
1. [Usage](#usage)
1. [REST API](#rest-api)
1. [Known limitations](#known-limitations)

## Description

`nvidia-docker-plugin` is a [Docker plugin](https://docs.docker.com/engine/extend/plugins/) designed to ease the process of deploying GPU-aware containers in heterogeneous environments. It acts as a daemon process, discovering host driver files and GPU devices and answers to volume mount requests originating from the Docker daemon.

The plugin also provides a [REST API](#rest-api) that one can query to get GPU information remotely or generate Docker arguments given volume names and device numbers.

## Usage

The plugin daemon can be tuned using the following arguments:

```sh
  -l string
        Server listen address (default "localhost:3476")
  -s string
        NVIDIA plugin socket path (default "/run/docker/plugins")
  -v string
        Path where to store the volumes (default "/var/lib/nvidia-docker/volumes")
```

Once the plugin is running, `nvidia-docker` will be able to connect to it and request information for containerization.
In case you upgrade the NVIDIA drivers, you will need to restart the plugin.

By default, `nvidia-docker-plugin` requires root privileges to create the Docker plugin socket.  
However, you can run it in an _almost_ unprivileged fashion by doing the following as root:

```sh
# Add a system user nvidia-docker
adduser --system --home /var/lib/nvidia-docker nvidia-docker
# Register the plugin with the Docker daemon
mkdir -p /etc/docker/plugins && echo "unix:///var/lib/nvidia-docker/nvidia-docker.sock" > /etc/docker/plugins/nvidia-docker.spec
# Set the mandatory permission
setcap cap_fowner+pe /usr/bin/nvidia-docker-plugin

# Run nvidia-docker-plugin as the nvidia-docker user
sudo -u nvidia-docker nvidia-docker-plugin -s /var/lib/nvidia-docker
```

## REST API

By default, `nvidia-docker-plugin` serves its REST interface locally on port `3476`. This will effectively prevent `nvidia-docker` from accessing the machine remotely through the `http` protocol (see [remote deployments](Using nvidia-docker#running-it-remotely)). This behavior can be changed using the `-l` [usage option](#usage) (e.g `-l :1234` for the wildcard address port `1234`).

REST endpoints are described below. Note that if a given client only cares about the latest API version, the version prefix can be omitted (i.e. `/gpu/info` is currently equivalent to `/v1.0/gpu/info`)

#### Version 1.0

* GET `/v1.0/gpu/info`, `/v1.0/gpu/info/json` 
> Queries the GPU devices information (akin to `nvidia-smi -q`).  
> Response format is `text/plain` and `application/json` respectively.

* GET `/v1.0/gpu/status`, `/v1.0/gpu/status/json`
> Queries the GPU devices status (akin to `nvidia-smi`).  
> Response format is `text/plain` and `application/json` respectively.

* GET `/v1.0/docker/cli`
> Queries the command-line parameters to use with `docker run` or `docker create`.  
> It accepts two query-string parameters: `dev` for devices (akin to `NV_GPU`) and `vol` for volumes.  
> This is useful if you don't want to rely on the `nvidia-docker` alternative CLI:  
> ```docker run -ti `curl -s http://localhost:3476/v1.0/docker/cli?dev=0+1\&vol=nvidia_driver` cuda```  

* GET `/v1.0/mesos/cli`
> Queries the command-line parameters to use when starting a Mesos agent.  
> Similar to `/v1.0/gpu/info/json` but compress and encodes the devices information in zlib / base64 (RFC 6920).

## Known limitations

1. NVIDIA driver installation needs to live on the same partition as the volume directory of the Docker plugin.  
`nvidia-docker-plugin` internally needs to create hard links to some driver files. Because of this, you need the NVIDIA driver (usually found under `/usr`) to be on the same partition as the Docker plugin volume directory (`/var` by default).
Possible workarounds includes installing your NVIDIA driver at a different location (see ``--advanced-options`` of the installer) or changing your plugin volume directory (see `nvidia-docker-plugin -v`)

**It is not recommended** to place the `nvidia-docker-plugin` volume directory under the same directory structure as the NVIDIA drivers (i.e. `/usr/lib`, `/usr/lib64` ...). This is due to the NVIDIA installer performing checks for conflicting files before performing an upgrade.