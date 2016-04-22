In this section, we will dive into the internals of nvidia-docker to explain the problems we are solving, and how. We will also present other options for solving these problems when possible. This should give you enough knowledge to be able to integrate NVIDIA GPU support for your own container deployment application, for instance if you don’t want to rely on the nvidia-docker CLI wrapper.  

If this is your goal, be aware that it's actually already possible to avoid relying on the `nvidia-docker` wrapper. Using the REST API of the `nvidia-docker-plugin` daemon you can launch GPU containers using only `docker`:
```
$ curl -s http://localhost:3476/v1.0/docker/cli
--device=/dev/nvidiactl --device=/dev/nvidia-uvm --device=/dev/nvidia3 --device=/dev/nvidia2 --device=/dev/nvidia1 --device=/dev/nvidia0 --volume-driver=nvidia-docker --volume=nvidia_driver_361.48:/usr/local/nvidia:ro
$ docker run -ti --rm `curl -s http://localhost:3476/v1.0/docker/cli` nvidia/cuda nvidia-smi
```
Another option is to leverage our [`nvidia` plugin](https://github.com/NVIDIA/nvidia-docker/tree/master/tools/src/nvidia) written in the Go programming language. Go is the language for containers so many projects should be able to reuse our code seamlessly.