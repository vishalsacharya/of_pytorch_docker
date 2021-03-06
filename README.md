# Docker + OpenFOAM&reg; + PyTorch

## Overview

The Dockerfile in this repository creates an image with [OpenFOAM-plus](https://openfoam.com/) and [PyTorch](https://pytorch.org/) support. The image is currently based on

- Ubuntu 20.04,
- OpenFOAM-v2006, and
- PyTorch 1.6 (only CPU).

There are also convenience scripts for creating and running a container based on the image. The *test* directory contains two examples demonstrating how to compile applications using *cmake* and *wmake*

OpenFOAM is not compiled from scratch but installed via the package manager ([read more](https://develop.openfoam.com/Development/openfoam/-/wikis/precompiled/debian)). Also for PyTorch, only the pre-compiled C++ part of the library, named *libtorch*, is contained on the image.

## Get in touch

If you would like to suggest changes or improvements regarding the

- build process,
- pre-installed packages,
- examples,
- documentation,
- ...

please use the [issue tracker](https://github.com/AndreWeiner/of_pytorch_docker/issues).

## How to use the image

Copy this repository and navigate into the top-level folder:
```
git clone https://github.com/AndreWeiner/of_pytorch_docker.git
cd of_pytorch_docker
```

The script *create_openfoam_container.sh* creates a container with suitable settings (e.g. mapping the user into the container, mounting the current directory, setting the working directory to *./test/*). The script also pulls the Docker image from [Dockerhub](https://hub.docker.com/repository/docker/andreweiner/of_pytorch) if it cannot be found locally. The default image and container names can be changed by passing them as command line arguments.

```
# default settings
./create_openfoam_container.sh

# use different image, e.g., to use an older version
./create_openfoam_container.sh "andreweiner/of_pytorch:of2006-py1.6-cpu" "of2006-py1.6-cpu" 
```

The *start_openfoam.sh* script starts an interactive shell instance in the running container. If you modified the container name in the previous step, provide the modified name as command line argument.

```
# default
./start_openfoam.sh

# custom container name
./start_openfoam.sh "of2006-py1.6-cpu"
```

The container's entry point is set to the *test* directory. There you are presented with two examples:

- **tensorCreation**: PyTorch tensor basics; compiled with **wmake**
- **simpleMLP**: implementation of a simple *multilayer perceptron* (MLP) class; compiled with **cmake**

### *tensorCreation*

To compile and run *tensorCreation*, execute:
```
# you must be inside the container for this to work
# execute ./start_openfoam.sh to launch a shell instance
cd tensorCreation
wmake
./tensorCreation
```

### *simpleMLP*

To compile and run *simpleMLP*, execute:
```
# you must be inside the container for this to work
# execute ./start_openfoam.sh to launch a shell instance
cd simpleMLP
mkdir build
cd build
cmake ..
make
./simpleMLP
```

## How to build the image

### Dockerfile using pre-compiled packages (default)

To build the image yourself, copy this repository and navigate into the top-level folder:
```
git clone https://github.com/AndreWeiner/of_pytorch_docker.git
cd of_pytorch_docker
```
If you want to upload the image to a Docker registry, consider the following naming convention when running the build command:
```
docker build -t user_name/of_pytorch:of2006-py1.6-cpu -f Dockerfile .
```

### Older versions of the Dockerfile

There are two more Dockerfiles which were used to build previous versions of the Docker image. They remain part of the repository since they might be helpful to some users.

- *Dockerfile.abi_source*: OpenFOAM is compiled from scratch; only some third-party packages are installed and, hence, some applications are missing
- *Dockerfile.no_abi*: OpenFOAM is compiled from scratch with modified compiler flags to be compatible with *libtorch* versions prior to version 1.3

By default, the OpenFOAM library will be compiled running two jobs in parallel. If you prefer to use more jobs, set the *NP* build argument, e.g.:
```
docker build --build-args NP=8 -t user_name/of_pytorch:of1912-py1.5-cpu -f Dockerfile.abi .
```
I also recommend to save the Docker output in a log-file:
```
docker build --build-args NP=8 -t user_name/of_pytorch:of1912-py1.5-cpu -f Dockerfile.abi . &> log.docker
```

### ABI

Since version 1.3 of PyTorch, there is a version of libtorch compiled with ABI enabled ([read more](https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_dual_abi.html)). The only change in prior versions compared to a regular compilation is the additional flag
```
-D_GLIBCXX_USE_CXX11_ABI=0
```
when compiling OpenFOAM. The flag is related to backwards compatibility for standards older than C++11 ([read more](https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_dual_abi.html)).

### Libtorch

The PyTorch library files are located in */opt/libtorch*. The environment variable **TORCH_LIBRARIES** can be used to indicate the location of certain header and library files to the compiler and linker. To compile PyTorch C++ code using *wmake*, add

```
EXE_INC = \
    -I$(TORCH_LIBRARIES)/include \
    -I$(TORCH_LIBRARIES)/include/torch/csrc/api/include \
```
to the include paths, and
```
EXE_LIBS = \
    -Wl,-rpath,$(TORCH_LIBRARIES)/lib $(TORCH_LIBRARIES)/lib/libtorch.so $(TORCH_LIBRARIES)/lib/libc10.so \
    -Wl,--no-as-needed,$(TORCH_LIBRARIES)/lib/libtorch_cpu.so \
    -Wl,--as-needed $(TORCH_LIBRARIES)/lib/libc10.so \
    -Wl,--no-as-needed,$(TORCH_LIBRARIES)/lib/libtorch.so
```
to the library paths.



