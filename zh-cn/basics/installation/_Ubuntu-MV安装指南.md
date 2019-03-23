
# Ubuntu安装

## 环境说明
 
 本文档提供的安装方式仅支持Ubuntu系统版本` 16.04`，且其CPU架构在2.0及以上，若CPU架构在2.0以下需要以源码方式安装！其他Ubuntu版本安装方式会尽快推出！

## 安装方式

- [PPA源](#PPA源安装)
- [二进制包](#二进制包安装)
- [Debian](#Debian安装)
- [源码](#源码编译安装)

> **注意：在安装或编译带`MPC`及`VC`功能的`PlatON`客户端之前，需要安装好`MPC`及`VC`相关依赖。**


## PPA源安装

添加`PPA`源并安装`PlatON`客户端，安装方式如下：

```bash
# 添加PPA
$ sudo add-apt-repository ppa:platonnetwork/platon
$ sudo apt-get update

# 安装带MPC及VC功能的Platon包
$ sudo apt-get install platon-all
```

通过`PPA`源安方式安装，会自动安装`MPC`及`VC`相关依赖。安装完成后，可执行程序将安装到： `/usr/bin/`。


## 二进制包安装

### 1. 安装依赖

 请参考[这里](#依赖包安装)安装`MPC`及`VC`相关依赖包。

### 2.安装PlatON客户端
```bash
# 下载
$ wget https://download.platon.network/0.5/platon-ubuntu-amd64-0.5.0-with-mv.tar.gz

# 解压
$ tar -xvzf platon-ubuntu-amd64-0.5.0-with-mv.tar.gz
```
解压内容如下：

- `platon`  PlatON客户端
- `ethkey`  公私钥对生成工具
- `ctool`   合约发布调用工具


## Debian包安装

### 1. 安装依赖

 请参考[这里](#依赖包安装)安装`MPC`及`VC`相关依赖包。

### 2.安装PlatON客户端

```bash

# 下载PlatON安装包 
$ wget https://download.platon.network/0.5/platon-all-ubuntu-amd64-0.5.0.deb

# 安装
$ sudo dpkg -i platon-all-ubuntu-amd64-0.5.0.deb
```

安装完成后，可执行程序将安装到： `/usr/bin/`。


## 源码编译安装

**Ubuntu编译环境要求**：

- 系统版本：`Ubuntu 16.04.1 及以上`，
- git：`2.19.1及以上`
- 编译器：`gcc(4.9.2+)`
- go语言开发包：`go(1.7+)`
- cmake:`3.0+` 

> **注意：保证满足编译环境要求**

执行以下步骤完成编译安装：

### 1. 安装依赖

 请参考[这里](#依赖包安装)安装`MPC`及`VC`相关依赖包。

### 1. 获取源码

```
$ git clone https://github.com/PlatONnetwork/PlatON-Go.git --recurive
```

### 2. 编译

  `MPC`功能和`VC`功能可以相互独立安装，以下提供三种编译方式，用户可以按需选择。

> **注意：编译带`MPC`或`VC`功能的`PlatON`源码之前需要确保环境已经安装好相关依赖，安装方式请参考[这里](#依赖安装)**。

- **编译带`MPC`功能`PlatON`客户端**

```bash
$ cd PlatON-Go
$ find ./build -name "*.sh" -exec chmod u+x {} \;
$ make all-with-mpc
```

>**提示**：`MPC`计算功能是`PlatON`平台实现隐私计算提供的基础设施。更多MPC相关请参考[这里](zh-cn/development/[Chinese-Simplified]-%e9%9a%90%e7%a7%81%e5%90%88%e7%ba%a6%e5%bc%80%e5%8f%91%e6%8c%87%e5%8d%97)。

- **编译带`VC`功能`PlatON`客户端**

```bash
$ cd PlatON-Go
$ find ./build/ -name "*.sh" -exec chmod u+x {} \;
$ make all-with-vc
```

>**提示**：`VC`验证计算功能是`PlatON`平台实现隐私计算提供的基础设施。更多VC相关请参考[这里](zh-cn/development/[Chinese-Simplified]-可验证合约)。

- **编译带`MPC`及`VC`功能`PlatON`客户端**
 
```bash
$ cd PlatON-Go
$ find ./build/ -name "*.sh" -exec chmod u+x {} \;
$ make all-with-mv
```

编译完成之后在`PlatON-Go/build/bin`目录下会生成`platon`、`ethkey`和`ctool`可执行文件，将此三个可执行文件拷贝到自己工作目录运行即可。

> 注意：重复编译会覆盖之前生成的可执行文件。

## 依赖包安装

### **安装`MPC`依赖**
  
以下提供三种`MPC`依赖安装方式，分别为二进制包安装、Debian包安装和编译`MPC`库源码安装，其中二进制包和Debian包的安装方式适用于CPU架构在0.2及以上Ubuntu16.04系统，若CPU架构在0.2及以下，需要自行编译`MPC`库源码安装！

- **二进制包安装方式：**

```bash
# 下载MPC依赖库
$ wget https://download.platon.network/0.5/platon-mpclib-ubuntu-amd64-0.5.0.tar.gz

# 解压
$ tar -xvzf platon-mpclib-ubuntu-amd64-0.5.0.tar.gz

$ mv platon-mpclib-ubuntu-amd64-0.5.0 platon-mpclib

#将解压后的该路径添加到环境变量
$ grep "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/platon-mpclib" ~/.bashrc || echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/platon-mpclib" >> ~/.bashrc

$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/home/path/to/platon-mpclib
```

- **Debian包安装方式:**

```
# 下载MPC lib安装包 
$ wget https://download.platon.network/0.5/platon-mpclib-ubuntu-amd64-0.5.0.deb

# 安装MPC lib
$ sudo dpkg -i platon-mpclib-ubuntu-amd64-0.5.0.deb
```

- **编译MPC库源码安装** 

编译方式请参考[隐私合约虚拟机编译](https://github.com/PlatONnetwork/privacy-contract-vm#building--installing)。
假定编译目录为`~/home/path/to/mpcvm/build`，编译将输出`MPC`依赖库到`~/home/path/to/mpcvm/build/lib`，将该路径添加到当前用户环境变量:

```bash
$ grep "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/mpcvm/build/lib" ~/.bashrc || echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/mpcvm/build/lib" >> ~/.bashrc

$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/home/path/to/mpcvm/build/lib
```

### **安装`VC`依赖**

```
$ sudo apt-get install libboost-all-dev 
$ sudo apt-get install llvm-6.0-dev llvm-6.0 libclang-6.0-dev 
$ sudo apt-get install libgmpxx4ldbl libgmp-dev libprocps4-dev libssl-dev
```