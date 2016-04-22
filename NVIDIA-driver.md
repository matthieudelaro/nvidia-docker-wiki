## Challenges
In order to execute a GPU application on your machine, you need to have the NVIDIA driver installed. The NVIDIA driver is composed of multiple kernel modules:
```
$ lsmod | grep nvidia
nvidia_uvm            711531  2 
nvidia_modeset        742329  0 
nvidia              10058469  80 nvidia_modeset,nvidia_uvm
```
It also provides a collection of user-level driver libraries that enable your application to communicate with the kernel modules and therefore the GPU devices:
```
$ ldconfig -p | grep -E 'nvidia|cuda'
       libnvidia-ml.so (libc6,x86-64) => /usr/lib/nvidia-361/libnvidia-ml.so
       libnvidia-glcore.so.361.48 (libc6,x86-64) => /usr/lib/nvidia-361/libnvidia-glcore.so.361.48
       libnvidia-compiler.so.361.48 (libc6,x86-64) => /usr/lib/nvidia-361/libnvidia-compiler.so.361.48
       libcuda.so (libc6,x86-64) => /usr/lib/x86_64-linux-gnu/libcuda.so
[...]
```
Notice how the libraries are tied to the driver version.  
The driver installer also provides utility binaries such as `nvidia-smi` and `nvidia-modprobe`. 

One of the early idea for containerizing GPU applications was to install the user-level driver libraries inside the container (for instance using option `--no-kernel-module` from the driver installer) . However,  the user-level driver libraries are tied to the version of the kernel module and all Docker containers share the host OS kernel. The version of the kernel modules had to match **exactly** (major and minor version) the version of the user-level libraries. Trying to run a container with a mismatched environment would immediately yield an error inside the container:
```
# nvidia-smi 
Failed to initialize NVML: Driver/library version mismatch
```
This approach made the images non-portable, making image sharing impossible and thus defeating of the main advantage of Docker. The solution is to make the images agnostic of the driver version. The Docker images we provide on the DockerHub are generic, but when creating a container the environment must be specialized for the host kernel module by mounting the user-level libraries from the host using the [`--volume`](https://docs.docker.com/engine/reference/run/#volume-shared-filesystems) argument of `docker run`.

The NVIDIA driver supports multiple host OSes, there are multiple ways to install the driver (e.g. runfile or deb/rpm package) and the installer can also be customized. Across distributions, there is therefore no portable location for the driver files. The list of files to import can also depend on your driver version.

## nvidia-docker
Our approach is equivalent to running `ldconfig -p` as shown above: we programatically parse the ldcache file (`/etc/ld.so.cache`) to discover the location of a predefined list of [ libraries](https://github.com/NVIDIA/nvidia-docker/blob/93bb65de7fc349e6de9f27abdaa75875f5572b17/tools/src/nvidia/volumes.go#L118-L168).
Some libraries can be found multiple times on the system, and some of them must **not** be picked. For instance, this is true for OpenGL libraries which can be provided by multiple vendors. We have a [function](https://github.com/NVIDIA/nvidia-docker/blob/93bb65de7fc349e6de9f27abdaa75875f5572b17/tools/src/nvidia/volumes.go#L173-L211) to blacklist libraries that are known to be provided by multiple sources.

Since those libraries can be scattered across the host filesystem, we create a Docker [named volume] (https://docs.docker.com/engine/userguide/containers/dockervolumes/) composed of hard links to the discovered libraries, we also have a copy fallback path if hard linking is not possible.
This volume can be managed by using the `nvidia-docker-plugin` daemon which implements the Docker API for [volume plugins](https://docs.docker.com/engine/extend/plugins_volume/):
```
$ docker volume inspect nvidia_driver_361.48 
[
    {
        "Name": "nvidia_driver_361.48",
        "Driver": "nvidia-docker",
        "Mountpoint": "/var/lib/nvidia-docker/volumes/nvidia_driver/361.48",
        "Labels": null
    }
]
```
The `nvidia-docker` wrapper will automatically add the volume arguments to the command-line before passing control to `docker`, you only need to have the `nvidia-docker-plugin` daemon running.

## Alternatives
If you don’t want to use the `nvidia-docker` wrapper, you can add the command-line arguments manually:
```
$ docker run --volume-driver=nvidia-docker --volume=nvidia_driver_361.48:/usr/local/nvidia:ro
````
To avoid using `--volume-driver` (since you can only use it once per command-line), you can create the named volume manually:
```
$ docker volume create --name=nvidia_driver_361.48 -d nvidia-docker
```
The two solutions above still require using `nvidia-docker-plugin` but this should be less of a problem since volume plugins are officially supported by Docker.

If you don’t want to use the volume plugin, you will have to locate the driver files manually using `ldconfig -p` or by parsing the ldcache. If your deployment environment is **entirely** homogeneous, you could simply hardcode the paths of the files for your use-case. For validation, you can peek at the volume created by `nvidia-docker-plugin`:
```
$ ls -R `docker volume inspect -f "{{ .Mountpoint }}" nvidia_driver_361.48`
``` 
We don’t recommend solutions based on locating the libraries through `find` since you might pick stray libraries from an older driver install.  
We recommend to suffix the name of the volume with the driver version, this would prevent mayhem if you update your driver but forget to recreate a new volume.
