A few examples of Dockerfiles are provided in the `samples/` folder, these images can be used to quickly test `nvidia-docker` on your machine. The samples are not available on the Docker Hub, you will need to build the images [locally](Building-images-locally):
```sh
make samples
```

Use `nvidia-docker` to test the samples:
```
nvidia-docker run sample:nvidia-smi
nvidia-docker run sample:vectorAdd
nvidia-docker run sample:deviceQuery
nvidia-docker run sample:matrixMulCUBLAS
nvidia-docker run sample:bandwidthTest
```
 
Here is a possible output for a container based on the `sample:deviceQuery` image:
```sh
./deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "GeForce GTX 980"
  [...]

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 7.5, CUDA Runtime Version = 7.5, NumDevs = 1, Device0 = GeForce GTX 980
Result = PASS