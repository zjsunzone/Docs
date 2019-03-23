## Installation on Ubuntu

This document is a PlatON installation guide on Ubuntu and is available in four installations.

- [PPA](#PPA-based-Installation)
- [Binary package](#Binary-package-based-Installation)
- [Debian package](#Debian-package-Installation)
- [Source code](#Source-code-based-Installation)

> **Note: **
> - `PlatON` with privacy and verifiable contracts can only be installed on Ubuntu 16.04. 
> - When installing through official binary packages, PPA sources and Debian packages, the native CPU architectures need to be above 2.0 and 2.0, otherwise PlatON needs to be installed by compiling and encoding.

### PPA based Installation

Add the `PPA` source and install the `PlatON` client as follows:

```bash
# Add PPA
$ sudo add-apt-repository ppa:platonnetwork/platon
$ sudo apt-get update

# Install Platon
$ sudo apt-get install platon-all
```

After the installation, the binaries and other components of the package should be installed to `/usr/bin`


### Binary package based Installation

#### 1. Dependencies Installation 
Please refer to [here](#Dependency package installation) to install related dependencies.

#### 2. PlatON Installaion

```bash
# Download
$ wget https://download.platon.network/0.5/platon-ubuntu-amd64-0.5.0-with-mv.tar.gz 

# Decompression  
$ tar -xvzf platon-ubuntu-amd64-0.5.0-with-mv.tar.gz 
```

The extracted files should be as following:

- `platon` client executable file 
- `ethkey` key generator 
- `ctool` wasm contract deploy kit 


### Debian package based Installation

#### 1. Dependencies Installation 

Please refer to [here](#Dependency-package-installation) to install related dependencies.

#### 2. PlatON Installation

```bash
# Download the `.deb` package
$ wget https://download.platon.network/0.5/platon-all-ubuntu-amd64-0.5.0.deb 

#installation
$ sudo dpkg -i platon-all-ubuntu-amd64-0.5.0.deb
```

After the installation, the binaries and other components of the package should be installed to `/usr/bin`

### Source code based installation

The Ubuntu build environment needs to meet the following requirements:

- System version: `Ubuntu 16.04.1 and above`
- git:`2.19.1 and above`
- Compiler: `gcc(4.9.2+)`
- go language development kit: `go(1.7+)`
- cmake:`3.0+`

Note: Make sure the compilation environment requirements are met！

The PlatON compilation and installation process is as follows:

#### 1. Dependencies Installation 

Please refer to [here](#Dependency-package-Installation) to install related dependencies.

#### 2. Get the source code

```
$ git clone https://github.com/PlatONnetwork/PlatON-Go.git --recursive
```

#### 3. Compilation 

```bash
$ cd PlatON-Go
$ find ./build/ -name "*.sh" -exec chmod u+x {} \;
$ make all-with-mv
```

After the compilation is completed, a series of executable files such as `platon` will be generated in the `PlatON-Go/build/bin` directory. You can copy the executable file to your working directory and run it as required.

### Dependency package Installation

#### 1. **Third-party dependencies installation**

```
$ sudo apt-get install libboost-all-dev
$ sudo apt-get install llvm-6.0-dev llvm-6.0 libclang-6.0-dev
$ sudo apt-get install libgmpxx4ldbl libgmp-dev libprocps4-dev libssl-dev
```
#### 2. **Extended library installation**
The following three installation modes are provided. The binary package and debian package installation modes are applicable to Ubuntu system with CPU architecture of 0.2 or above. If the CPU architecture is 0.2 or below, you need to compile source code yourself!

- **Binary package based installation**

```bash
# Download 
$ wget https://download.platon.network/0.5/platon-mpclib-ubuntu-amd64-0.5.0.tar.gz

# Decompression 
$ tar -xvzf platon-mpclib-ubuntu-amd64-0.5.0.tar.gz

$ mv platon-mpclib-ubuntu-amd64-0.5.0 platon-mpclib

#Add the extracted path to the environment variable
$ grep "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/platon-mpclib" ~/.bashrc || echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/platon-mpclib" > > ~/.bashrc

$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/home/path/to/platon-mpclib
```

- **Debian package based installation**

```
# Download lib installation package
$ wget https://download.platon.network/0.5/platon-mpclib-ubuntu-amd64-0.5.0.deb

# Install lib
$ sudo dpkg -i platon-mpclib-ubuntu-amd64-0.5.0.deb
```

- **Compile library source installation**

For the compilation method, please refer to [Privacy Contract Virtual Machine Compilation](https://github.com/PlatONnetwork/privacy-contract-vm#building--installing).
Assuming the build directory is `~/home/path/to/mpcvm/build`, the compiler will output the `MPC` dependency library to `~/home/path/to/mpcvm/build/lib`, adding the path to the current user. Environmental variables:

```bash
$ grep "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/mpcvm/build/lib" ~/.bashrc || echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/mpcvm/build /lib" >> ~/.bashrc

$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/home/path/to/mpcvm/build/lib
```

