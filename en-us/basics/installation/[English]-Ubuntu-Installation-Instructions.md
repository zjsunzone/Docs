## Installation on Ubuntu

This document is a PlatON installation guide on Ubuntu and is available in four installations.

- [PPA](#PPA-based-Installation)
- [Binary package](#Binary-package-based-Installation)
- [Debian package](#Debian-package-Installation)
- [Source code](#Source-code-based-Installation)

> **Note: **
> - The official binary package, PPA source and Debian package only support version 16.04 and above. 
> - When installing through official binary packages, PPA sources and Debian packages, the native CPU architectures need to be above 2.0 and 2.0, otherwise PlatON needs to be installed by compiling and encoding.

### PPA based Installation

Add the `PPA` source and install the `PlatON` client as follows. Note, there are some differences in installation methods between ubuntu18.04 and ubuntu16.04:

- Ubuntu16.04 

```bash
# Add PPA
$ sudo add-apt-repository ppa:platonnetwork/platon
$ sudo apt-get update

# Install Platon
$ sudo apt-get install platon-all
```
- Ubuntu18.04 

```bash
# 添加PPA
$ sudo add-apt-repository ppa:platonnetwork/platon
$ sudo vi platonnetwork-ubuntu-platon-bionic.list
>> Change bionic to xenial in the file

$ sudo apt-get update
# Install Platon
$ sudo apt-get install platon
```

After the installation, the binaries and other components of the package should be installed to `/usr/bin`


### Binary package based Installation

```bash
# Download
$ wget https://download.platon.network/0.5/platon-ubuntu-amd64-0.5.0.tar.gz

# 解压
$ tar -xvzf platon-ubuntu-amd64-0.5.0.tar.gz
```

The extracted files should be as following:

- `platon` client executable file 
- `ethkey` key generator 
- `ctool` wasm contract deploy kit 


### Debian package based Installation

```bash
# Download the `.deb` package
$ wget https://download.platon.network/0.5/platon-ubuntu-amd64-0.5.0.deb

#installation
$ sudo dpkg -i platon-ubuntu-amd64-0.5.0.deb
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


#### 1. Get the source code

```
$ git clone https://github.com/PlatONnetwork/PlatON-Go.git --recursive
```

#### 2. Compilation 

```bash
$ cd PlatON-Go
$ find ./build -name "*.sh" -exec chmod u+x {} \;
$ make all
```

After the compilation is completed, a series of executable files such as `platon` will be generated in the `PlatON-Go/build/bin` directory. You can copy the executable file to your working directory and run it as required.
