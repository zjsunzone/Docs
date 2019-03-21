# Setting up privacy computation environment

## Checking the CPU architecture

At present, the official only supports `Intel`2 generation and above `CPU` architecture. Before setting up the environment, it is necessary to check whether the machine meets the minimum requirements.

- **Linux platform**

Use the following command to view the `CPU` information:

```
$ cat /proc/cpuinfo
```

For example, the following information can be found in the output:

```
Model name : Intel(R) Xeon(R) CPU E5-2620 v2 @ 2.10GHz
```

Among them, `E5-2620` and `v2` both indicate that `CPU` is a second-generation architecture, so it meets the requirements.

- **Windows platform**

Right click ***My Computer*** with mouse, select ***Attribute***, for example to view the processor as:

```
Intel(R) Xeon(R) CPU E5-2620 v2 @ 2.10GHz
```

Similarly, `E5-2620` and `v2` both indicate that `CPU` is a second-generation architecture and therefore meets the requirements.

> **If the CPU architecture meets Intel v2 or above, you can continue with the steps below**



## Install JAVA development environment

 Install jdk1.8 or above, please refer to [oracle official website](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html), configure `CLASSPATH, PATH, JAVA_HOME` environment variables after installation.

## Installing maven

 Install maven 3.3.9 or above, refer to [maven official website](http://maven.apache.org/download.cgi).

## Building a PlatON Computing Network

- **Method 1: Connect to the testnet**
  1. Install the `platon` executable program locally, refer to [Installation Guide](en-us/basics/[English]-Installation-Instructions).
  2. Connect the compute node to the test network by referring to the [Test Network Connection Guide](en-us/basics/[English]-Connect-to-Test-Network).

- **Method 2: Build a private test network**
  Set up a private test network by yourself, as follows:

1. Install the `platon` executable with the `MPC` function and check the [Installation Guide](en-us/basics/[English]-Installation-Instructions).
2. Set up the cluster environment, check [PlatON cluster environment](en-us/basics/[English]-Private-Networks#PlatON-cluster).
3. Enable MPC calculation on the node, check [Enable MPC calculation function](en-us/basics/[English]-Private-Networks#Enabling-MPC-for-a-node).

## Install the compiler integration package

- **Linux platform**

Installation using binary package, currently only supports Ubuntu16.04 version, you can directly download [binary package](https://download.platon.network/0.5/platon-ubuntu-amd64-mpc-compiler.zip) to extract it. :

```bash
./
├── bin
│ ├── libprotobuf.so
│ ├── libprotoc.so
│ ├── plang
│ └── protoc
├── compile_sample.sh
├── config_sample.json
├── include
│ ├──batcher.h
│ ├── bit.h
│ ├── block.h
│ ├──similar.h
│ ├── constants.h
│ ├── emp-tool-miracl.def
│ ├── emp-tool-relic.def
│ ├── float_circuit.h
│ ├── google
│ │ └──protobuf
│ │ ├── any.h
│ │ ...
│ │ └── wrappers.proto
│ ├── integer.h
│ ├── number.h
│ └──swappable.h
├── lib
│ └── clang
│ └── 6.0.1
│ └── include
│ ├── adxintrin.h
...
│ ├── f16cintrin.h
...
├── readme.txt
├── SimpleAddProto.cpp
└── SimpleAddProto.protos
```

This integration package already includes: `protobuf` toolset, `emp-tool` library, etc., so there is no need to install these included components separately.

- **Windows platform**

Windows platform privacy compilation toolset, due to compiler version compatibility issues, currently can not provide compiler binary installation package, you need to refer to the next steps, separate [install privacy contract compiler](#Installing-the-Privacy-Contract-Compiler).

> `protobuf` toolset, privacy algorithm header file (`emp-tool` library) binary package, click here [download binary package](https://download.platon.network/0.5/platon-win-amd64-mpc-compiler.zip).

## Installing the Privacy Contract Compiler

- **Linux platform**

Ubuntu16.04 can be downloaded directly by downloading [Binary Package](https://download.platon.network/0.5/platon-ubuntu-amd64-mpc-compiler.zip).

Other versions of Linux use source code to compile and install, taking Ubuntu as an example:

```bash
#下载来源
git clone https://github.com/PlatONnetwork/privacy-contract-compiler.git
# Switch to the source directory
cd privacy-contract-compiler
#Create a build directory
mkdir build
#建编译工程
cmake .. && make -j4
#编译
Make
```

After the installation is successful, generate the `plang` compiler in the `build/bin` directory, and generate `libmpc-jit.so` related dynamic libraries and links in the `build/lib` directory.

- **Windows platform**

Currently only source code compilation is supported, and it is required to follow the VS2015 and above compilers.

Compile steps:

```bash
#下载来源
git clone https://github.com/PlatONnetwork/privacy-contract-compiler.git
# Switch to the source directory
cd privacy-contract-compiler
#Create a build directory
mkdir build
#建编译工程
cmake .. && make -j4
```

After the `cmake` is built, generate the LLVM.sln VisualStudio solution file in the build directory, open the file and compile the project. After the compilation is complete, the privacy contract compiler `plang` will be generated in `Debug\bin` or `Release\bin`.


## Installing the protobuf toolset

PlatON privacy contract uses `protobuf` encoded parameters, and the `protobuf` is customized.

- Linux platform

  If it is Ubuntu16.04, you can download [binary package](https://download.platon.network/0.5/platon-ubuntu-amd64-mpc-compiler.zip) to extract it and use it.

  Other versions of Linux, using source installation:

```bash
# download github source
git clone https://github.com/PlatONnetwork/protobuf.git --recursive
#Switch working directory
cd protobuf
#Create a build directory
mkdir build
# cmake build
cd build && cmake ../cmake
# compile
make
```

The installation is successful, build executable program `protoc` in `build/bin`, `build/lib` production related dynamic library, the header file is under `protobuf/src`.

- Windows platform

  Binary installation:

    Win7, win8, win10 platform directly download [binary package](https://download.platon.network/0.5/platon-win-amd64-mpc-compiler.zip) decompression, in the bin directory: `protoc.exe`, `libprotoc.dll`, `libprotobuf.dll` are `protobuf` compilers and dynamic libraries.

  Source installation:

```bash
# download github source
git clone https://github.com/PlatONnetwork/protobuf.git --recursive
#Switch working directory
cd protobuf
#Create a build directory
mkdir build
# cmake build
cd build && cmake ../cmake
```

After the `cmake` is built, generate the protobuf.sln VisualStudio solution file in the build directory, open the file and compile the project. After the compilation is complete, generate the privacy contract compiler `protoc.exe` in the Debug\bin or Release\bin directory. `libprotoc.dll` and `libprotobuf.dll`.

## Installing the MPC header file library

> Use the binary package to install the privacy contract compilation toolset, you can skip this step.

If you use the source code to compile the toolset, you need to download the [header package](https://download.platon.network/0.5/platon-mpc-crossplat-headers.zip) directly, and extract it as follows:

```bash
Include/
├── batcher.h
├── bit.h
├── block.h
├──similar.h
├── constants.h
├── emp-tool.def
├── float_circuit.h
├── integer.h
├── number.h
└── swappable.h
├── google
| └──protobuf

```

When compiling the privacy contract by `plang`, you need to add the include path to the compile option `-I`.