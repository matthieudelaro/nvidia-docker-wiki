## Contents
1. [Challenges](#challenges)
1. [nvidia-docker](#nvidia-docker)
1. [Alternatives](#alternatives)

## Challenges
GPUs are exposed as separate device files in `/dev`,  along with additional devices:
```
$ ls -l /dev/nvidia*
crw-rw-rw- 1 root 195,   0 Apr 20 11:42 /dev/nvidia0
crw-rw-rw- 1 root 195,   1 Apr 20 11:42 /dev/nvidia1
crw-rw-rw- 1 root 195, 255 Apr 20 11:42 /dev/nvidiactl
crw-rw-rw- 1 root 247,   0 Apr 20 11:42 /dev/nvidia-uvm
```
Docker allows users to whitelist access to specific devices (using [cgroups](https://www.kernel.org/doc/Documentation/cgroup-v1/devices.txt)) when starting a container by passing the argument [`--device`](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) to `docker run`. On a multi-GPUs machine, a common use case is to launch multiple jobs in parallel, each one using a subset of the available GPUs. The most basic solution is to use the environment variable `CUDA_VISIBLE_DEVICES`, but this doesn’t guarantee proper isolation. By using device whitelisting in Docker, you can restrict which GPUs a container will be able to access. For instance, container A is granted access to `/dev/nvidia0` while container B is granted access to `/dev/nvidia1`. Devices `/dev/nvidia-uvm` and `/dev/nvidiactl` do not correspond to a GPU and they must be accessible for all containers.

The first challenge is to map the device files (or in other words, the minor number of the device) ordering to the PCI bus ordering (as reported by `nvidia-smi`). This is important when you have different models of GPUs on your machine and you want to assign a container to one GPU in particular. The GPU numbering reported by `nvidia-smi` doesn’t always match the minor number of the device file:
```
$ nvidia-smi -q
GPU 0000:05:00.0
 Minor Number: 3

GPU 0000:06:00.0
 Minor Number: 2
```

The second challenge is related to the `nvidia_uvm` kernel module, it is not loaded automatically at boot time,  thus `/dev/nvidia-uvm` is not created and the container might have insufficient permission to load the kernel module itself. The kernel module must be manually loaded before starting any CUDA container.

## nvidia-docker
GPUs are enumerated using function `nvmlDeviceGetCount` from the [NVML library](https://developer.nvidia.com/nvidia-management-library-nvml) and the corresponding device minor is obtained with the function `nvmlDeviceGetMinorNumber`. If the device minor number is N, the device file is simply /dev/nvidiaN.
Isolation is controlled using the environment variable `NV_GPU`, by passing the indices of the GPUs to isolate, for instance:
```
$ NV_GPU=0,1 nvidia-docker run -ti nvidia/cuda nvidia-smi
```
The `nvidia-docker` wrapper will find the corresponding device files and add the `--device` arguments to the command-line before passing control to `docker`.

To manually load `nvidia_uvm` and create `/dev/nvidia-uvm`, we use the command `nvidia-modprobe -u -c=0` on the host when starting the `nvidia-docker-plugin` daemon.

## Alternatives
If you don’t want to use the `nvidia-docker` wrapper, you can add the command-line arguments manually:
```
$ docker run --device=/dev/nvidiactl --device=/dev/nvidia-uvm --device=/dev/nvidia0
````
This needs to be used in combination with the command-line arguments for mounting the volume containing the the user-level driver libraries.
Listing the available GPUs can be done using `nvmlDeviceGetCount` from NVML or `cudaGetDeviceCount` from the CUDA API. We recommend using NVML since it also provides `nvmlDeviceGetMinorNumber` to find the device file to mount. Having a correct mapping between the device file and the isolated GPU is essential if you want to gather utilization metrics using NVML or nvidia-smi. If you still want to use the CUDA API, be sure to unset the environment variable `CUDA_VISIBLE_DEVICES`, otherwise some GPUs on the system might not be listed.

To load `nvidia_uvm`, you should also use `nvidia-modprobe -u -c=0` if available. If it’s not, you need to do the `mknod` [manually](http://docs.nvidia.com/cuda/cuda-getting-started-guide-for-linux/#runfile-verifications).
