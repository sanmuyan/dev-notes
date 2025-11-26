# CUDA 驱动相关

一般情况说的 CUDA 驱动是指 CUDA Toolkit 工具包，CUDA Toolkit 工具包是 GPU 加速应用的开发环境，要使用 CUDA 要先安装 GPU 驱动

## CUDA Toolkit 安装

[CUDA Toolkit](https://developer.nvidia.com/cuda-downloads)

### Ubuntu

```shell
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
# 开源驱动
sudo apt-get install -y nvidia-open
# 专用驱动
# sudo apt-get install -y cuda-drivers
# CUDA Toolkit 工具包
sudo apt-get -y install cuda-toolkit-13-0

nvidia-smi
nvcc --version
```

### WSL-Ubuntu

WSL 不用安装 Linux GPU 驱动，只需要在宿主机安装 Window 驱动，驱动会自动集成到 WSL

```shell
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-toolkit-13-0
```

## Docker 支持

要在容器使用运行 CUDA 应用，nvidia-container-toolkit 工具把 GPU 映射到容器内。注意 nvidia-docker2 已经过时。

1. 安装 nvidia-container-toolkit，CUDA Toolkit 仓库应该已经包含 nvidia-container-toolkit，如果找不到包参考 [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
```shell
export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.18.0-1
sudo apt-get install -y \
      nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
      libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION} 
```
2. 配置 Docker
```shell
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
cat /etc/docker/daemon.json
{
    "runtimes": {
        "nvidia": {
            "args": [],
            "path": "nvidia-container-runtime"
        }
    }
}   
```
3. 测试
```shell
docker run --gpus all nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -gpu -benchmark

...
30720 bodies, total time for 10 iterations: 34.513 ms
= 273.439 billion interactions per second
= 5468.787 single-precision GFLOP/s at 20 flops per interaction
```
4. 运行应用，可直接使用 `nvidia/cuda:13.0.0-devel-ubuntu22.04` 镜像