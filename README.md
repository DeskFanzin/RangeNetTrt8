 RangeNetTrt8
# IMPORTANT NOTE! #
. **This is a fork for using rangenet_lib with SuMa++** As Suma++ uses old tensorrt versions, I found that upgrading the dependencies would be useful. So here it is, a new version of rangenet_lib especifically for usage with SuMa++. Most of the work was done by natsu-akatsuki, the original owner of this repo. Please be aware that ROS or any functions besides infer example are not implemented!


## Purpose

1. **Use more newer dependencies and APIs**. Specifically, we deploy the [RangeNet repository](https://github.com/PRBonn/rangenet_lib) in an environment with TensorRT 8+, Ubuntu 20.04+, remove Boost dependency, manage TensorRT objects and GPU memory with smart pointers, and provide ROS demo.

3. <b>Faster Performance</b>. Resolve the issue of reduced segmentation accuracy when using FP16 ([issue#9](https://github.com/PRBonn/rangenet_lib/issues/9)), achieving a significant speed boost without sacrificing accuracy. Preprocess data using CUDA. Perform KNN post-processing with libtorch (refer to [here](https://github.com/PRBonn/lidar-bonnetal/blob/master/train/tasks/semantic/postproc/KNN.py)).

<p align="center">
	<img src="assets/000000.png" alt="img" width=50% height=50% />
</p>

## Prerequisites

### Step 1: Download and Extract libtorch

> **Note**  
> Using the Torch library from Conda was observed to slow down the post-processing stage from 6 ms to 30 ms.

```bash
$ wget -c https://download.pytorch.org/libtorch/cu113/libtorch-cxx11-abi-shared-with-deps-1.10.2%2Bcu113.zip -O libtorch.zip
$ unzip libtorch.zip
```

Step 2: Set up the deep learning environment (install NVIDIA driver, CUDA, TensorRT, cuDNN). The tested configurations are listed below. At least <u>3000 MB</u> of GPU memory is required.

| Ubuntu |           GPU           | TensorRT |      CUDA       |    cuDNN    |         —          |
|:------:|:-----------------------:|:--------:|:---------------:|:-----------:|:------------------:|
| 20.04  |        TITAN RTX        |  8.2.3   | CUDA 11.4.r11.4 | cuDNN 8.2.4 | :heavy_check_mark: |
| 20.04  | NVIDIA GeForce RTX 3060 | 8.4.1.5  | CUDA 11.3.r11.3 | cuDNN 8.0.5 | :heavy_check_mark: |
| 22.04  | NVIDIA GeForce RTX 3060 | 8.2.5.1  | CUDA 11.3.r11.3 | cuDNN 8.8.0 | :heavy_check_mark: |
| 22.04  | NVIDIA GeForce RTX 3060 | 8.4.1.5  | CUDA 11.3.r11.3 | cuDNN 8.8.0 | :heavy_check_mark: |
| 22.04  | NVIDIA GeForce RTX 4070 | 8.4.1.5  | CUDA 11.7.r11.7 | cuDNN 8.8.0 | :heavy_check_mark: | 


Add the following environment variables to ~/.bashrc:

```bash
# Example configuration:

# >>> Deep Learning Configuration >>>
# Import CUDA environment
CUDA_PATH=/usr/local/cuda/bin
CUDA_LIB_PATH=/usr/local/cuda/lib64

# Import TensorRT environment
export TENSORRT_DIR=${HOME}/Application/TensorRT-8.4.1.5/
TENSORRT_PATH=${TENSORRT_DIR}/bin
TENSORRT_LIB_PATH=${TENSORRT_DIR}/lib

# Import libtorch environment
export Torch_DIR=${HOME}/Application/libtorch/share/cmake/Torch

export PATH=${PATH}:${CUDA_PATH}:${TENSORRT_PATH}
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:${CUDA_LIB_PATH}:${TENSORRT_LIB_PATH}
```

Step 3: Install apt and Python packages

```bash
$ sudo apt install build-essential python3-dev python3-pip apt-utils git cmake libboost-all-dev libyaml-cpp-dev libopencv-dev python3-empy
$ pip install catkin_tools trollius numpy
```

## Install

Step 1: Clone the Repository

```bash
$ git clone https://github.com/DeskFanzin/RangeNetTrt8 ~/rangetnet_ws/src
```

</details>

## Usage

The first run may take some time to generate the TensorRT optimized engine.


## FAQ

<details> 
    <summary>:question: <b>Issue 1:</b> 
        [libprotobuf ERROR google/protobuf/text_format.cc:298] Error parsing text-format onnx2trt_onnx.ModelProto: 1:1:
    </summary>

The ONNX model is incomplete. Re-download the model.

</details> 

<details> 
    <summary>:question: <b>Issue 2:</b> 
        Abnormal prediction results when upgrading TensorRT from 8.2 to 8.4. See <a href="https://github.com/Natsu-Akatsuki/RangeNetTrt8/issues/8">issue#8</a>.
    </summary>

Skip optimization of weights in layer 235.

</details> 

<details> 
    <summary>:question: <b>Issue 3:</b> 
        error: A __device__ variable cannot be marked constexpr
    </summary>

The CUDA version is too low. Upgrade CUDA (issue#4). For lower versions like CUDA 11.1, refer to issue#2.

</details> 

<details> 
    <summary>:question: <b>Issue 4:</b> 
        Segmentation fault when visualizing single point cloud frames in Ubuntu 22.04 using PCL
    </summary>

Use PCL library version 1.13.0+. See more in [Here](https://github.com/PointCloudLibrary/pcl/pull/5252).

</details>
