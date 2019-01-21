**This document provides `PlatON` installation guide for different environments， After installation, you can refer to the document [Private Network](https://github.com/PlatONnetwork/wiki/wiki/%5BEnglish%5D-Private-Networks) to start.**

Follow the appropriate link below to find installation instructions for your platform.
+ Installation Instructions for Linux/Unix
  - [Ubuntu](#Installing-on-Ubuntu)
+ [Installation Instructions for Windows](#Installing-on-Windows)
 
## Installing on Ubuntu

The Ubuntu runtime environment needs to meet the following requirements:
- System version: `Ubuntu 16.04.1 or above`

There are four ways of installation on Ubuntu: 
- binaries package
- PPA source
- debian package
- source code

### Binary package based installation

The official binary package file download link for ubuntu is: [https://download.platon.network/platon-ubuntu-amd64-bin.tar.gz](https://download.platon.network/platon-ubuntu-amd64-bin.tar.gz)

```bash
$ wget https://download.platon.network/platon-ubuntu-amd64-bin.tar.gz
$ tar -xvzf platon-ubuntu-amd64-bin.tar.gz
```
The extracted files should be as following:
- `platon` client executable file
- `ethkey` key generator

### PPA based installation

Add PPA to your system and update：

```
# add PPA
$ sudo add-apt-repository ppa:platonnetwork/platon
$ sudo apt-get update

# install platon
$ sudo apt-get install platon-all
```

After the installation, the binaries and other components of the package should be installed to `/usr/bin/`

### Debian package based installation

Download the `.deb` package and then install.

```bash
# download
$ wget https://download.platon.network/platon-all-ubuntu-amd64.deb

# install
$ sudo dpkg -i platon-all-ubuntu-amd64.deb
```

After the installation, the binaries and other components of the package should be installed to `/usr/bin/`

### Source code based installation

The Ubuntu build environment needs to meet the following requirements:
- System version: `Ubuntu 16.04.1 or above`
- git：`2.19.1 or above`
- Compiler: `gcc(4.9.2+)`
- go language development kit: `go(1.7+)`

>**Note**: Make sure the compilation environment requirements are met！

The PlatON compilation and installation process is as follows:

#### 1. Clone `platon` source code to local target folder:

```
$ git clone https://github.com/PlatONnetwork/PlatON-Go.git
```

#### 2. Compilation

##### Compiling `Platon` without `mpc` capability by default

```bash
$ cd PlatON-Go
$ find ./build -name "*.sh" -exec chmod u+x {} \;
$ make all
```

##### Compiling `Platon` with `mpc` capability 

To enable `MPC` function on `platon`, compile `MPC VM` module and link it to the platon. Steps are as follows:

- Compiling `MPC VM`

Please referring to [Privacy Contract VM Compilation](https://github.com/PlatONnetwork/privacy-contract-vm#building--installing).
Assuming that the compilation directory is `home/path/to/mpcvm/build`, the compilation will output the `MPC VM` runtime libraries to `home/path/to/mpcvm/build/lib`.

- Setup environment

Append the compiled `MPC VM` libraries path `~/home/path/to/mpcvm/build/lib` to current user libraries environment variable:

```bash
grep "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/mpcvm/build/lib" ~/.bashrc || echo "export  LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/mpcvm/build/lib" >> ~/.bashrc

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/home/path/to/mpcvm/build/lib
```

- Compiling `platon`

```bash
$ cd PlatON-Go
$ find ./build -name "*.sh" -exec chmod u+x {} \;
$ make all-with-mpc
```
After compilation, the `platon`and `ethkey` executable files will be generated in the `PlatON-Go/build/bin` directory.

>**Hint**：
>MPC is secure multi-party computing feature supported by the Platon platform for privacy calculations. **Only Ubuntu supported now**. For more information about MPC, please refer [Reference](https://github.com/PlatONnetwork/wiki/wiki/%5BEnglish%5D-PlatON-Privacy-Contract-Guide)


## Installing on Windows

The Windows environment supports three installation modes:
- Binary package
- Chocolatey installation
- source code

### Binary package based installation

Windows version of the Platon binary download link is: [https://download.platon.network/platon-windows-amd64-bin.zip](https://download.platon.network/platon-windows-amd64-bin.zip) download. No installation is required after downloading, and it can be used directly by decompression.

The extracted files should be as following:
- `platon` client executable file
- `ethkey` key generator

### Installation via Chocolatey

Start PowerShell as an administrator and install Platon using the choco command:

```
choco install platon --version=0.2.0
```

You will find `platon`,`ethkey` in the default installation path `C:\ProgramData\chocolatey\bin`.

### Source code based installation

The Windows build environment requires:
- git:`2.19.1`
- go language development kit: `go(1.11+)`
- mingw: `mingw(V6.0.0)`

> Note: You can install the above compilation environment by yourself. Before compiling the `Platon` source code, make sure that the above environment works properly. The `chocolatey` installation can also be used, referring to the following ways:

#### 1. Install chocolatey 

 We use the Chocolatey package manager to install the required build tools. If you don't have Chocolatey, follow the instructions on [Chocolatey](https://chocolatey.org).

#### 2. Install compilation environment

Start PowerShell as an administrator and install Platon using the choco command:

```
// install git
choco install git
// install golang
choco install golang
// install mingw
choco install mingw
```

Most of the software installed with the Chocolatey package manager use the default installation path, although some publishers override the default settings in their software. Installing these packages will modify the PATH environment variable. The final installation path can be found in PATH. 

#### 3. Get `PlatON` source code

Create `src/github.com/PlatONnetwork/` and `bin` directories under the current `%GOPATH%` directory and clone the source code of `PlatON-GO` under the `PlatONnetwork` directory:

```
git clone https://github.com/PlatONnetwork/PlatON-Go.git
```

#### 4. Compile

Execute the compilation command under the source directory `PlatON-GO`, as follows:

```
go run build/ci.go install ./cmd/platon
```

After compilation, the `platon` and `ethkey` executable files will be generated in the `PlatON-Go/build/bin` directory.



## Docker环境运行

#### docker安装

docker安装比较简单，国内的话我们使用[阿里云Docker镜像源](https://yq.aliyun.com/articles/110806)，国外使用[Docker官方镜像](https://docs.docker.com/install/linux/docker-ce/ubuntu/)。

#### 运行容器

(https://hub.docker.com/r/platonnetwork/platon)

- 拉取platon镜像
 
```bash
$ sudo docker pull platonnetwork/platon:tag #例如: platonnetwork/platon:0.3.0
```

- 运行platon容器

执行以下命令在容器中启动platon节点，其默认得RPC端口为6789，P2P端口为16789：

```bash
$ sudo docker run -d platonnetwork/platon:tag
```

若想开启端口映射，可执行以下命令：

```bash
$ sudo docker run -d -e PLATONIP="192.168.120.20" -p 6789:6789  -p 16789:16789 --name platon platonnetwork/platon:0.3.0
```

其中，`PLATONIP`为本机服务器地址。

- 进入容器:

```bash
$ sudo docker exec -it container_name bash #container_name为容器id或者名字
```

执行一下命令进入platon的js终端(进入容器后默认目录就在/opt/node下):

```bash
# platon attach ipc:./data/platon.ipc
```

注意：若没有开通端口映射，只能在容器内进入以上命令进入js终端，开启端口映射之后可在本机通过以下命令进入js终端：

```bash
# platon attach http://PLATONIP:6789
```

- 服务启停命令

在容器内运行以下命令可控制`Platon`节点启动和停止

```bash
# supervisorctl start platon #启动platon
# supervisorctl stop platon #停止platon
# supervisorctl restart platon #重启platon
```

如果以上命令无效，请使用以下命令：

```bash
# /etc/init.d/supervisor stop
# /etc/init.d/supervisor start
```

**说明：容器中platon的数据目录位于`/opt/node`目录下，系统默认只有一个钱包账户，密码为:123456**


