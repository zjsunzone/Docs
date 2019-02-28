**本文档提供`PlatON`在不同环境的安装，安装完成之后需要启动节点请参考文档[私有网络](/zh-cn/basics/[Chinese-Simplified]-%E7%A7%81%E6%9C%89%E7%BD%91%E7%BB%9C)。**

PlatON支持不同的运行环境。
+ Linux/Unix环境安装
  - [Ubuntu](#Ubuntu安装)
+ [Windows环境安装](#Windows安装)
+ [Docker环境运行](#Docker环境运行)

## Ubuntu安装

Ubuntu环境要求：
- 系统版本：`Ubuntu 16.04.1 及以上`

Ubuntu环境支持以下四种安装方式： 

- 官方二进制包
- PPA源
- debian包
- 源码

### 官方二进制包下载

官方现提供两种二进制包，用户可按需下载安装，其下载地址如下：

- PlatON基础包：<https://download.platon.network/0.5/platon-ubuntu-amd64-0.5.0.tar.gz>
- 带有MPC及VC功能的PlatON包：<https://download.platon.network/0.5/platon-ubuntu-amd64-0.5.0-with-mv.tar.gz

1.安装PlatON基础包

```bash
# 下载
$ wget https://download.platon.network/0.5/platon-ubuntu-amd64-0.5.0.tar.gz

# 解压
$ tar -xvzf platon-ubuntu-amd64-0.5.0.tar.gz
```

解压内容如下：

- `platon` PlatON客户端
- `ethkey` 公私钥对生成工具
- `ctool`  合约发布调用工具

2.安装带有MPC及VC功能的PlatON二进制包

> 注意：
> 1. 官方提供的所有带MPC功能的二进制包只支持CPU架构在2.0及以上的主机上运行，若cpu架构在2.0以下需自行编译源码，编译方式请参考[编译带`MPC`功能`PlatON`客户端](#编译带MPC功能PlatON客户端)
> 2. 在安装所有带MPC功能的二进制包之前需要先安装MPC VM的依赖库，并配置环境变量，依赖库下载地址: <https://download.platon.network/0.5/platon-mpclib-ubuntu-amd64-0.5.0.tar.gz>。
> 3. 在安装所有带VC功能的二进制包之前需要先安装相关依赖。

- 安装`MPC VM`依赖

```bash
# 下载MPC VM依赖库
$ wget https://download.platon.network/0.5/platon-mpclib-ubuntu-amd64-0.5.0.tar.gz

# 解压
$ tar -xvzf platon-mpclib-ubuntu-amd64-0.5.0.tar.gz
```

将解压后的该路径添加到环境变量：

```bash
$ grep "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/platon-mpclib-ubuntu-amd64-0.5.0" ~/.bashrc || echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/platon-mpclib-ubuntu-amd64-0.5.0" >> ~/.bashrc

$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/home/path/to/platon-mpclib-ubuntu-amd64-0.5.0
```
- 安装`VC`依赖

```
# 安装依赖
$ sudo apt-get install libboost-all-dev 
$ sudo apt-get install llvm-6.0-dev llvm-6.0 libclang-6.0-dev 
$ sudo apt-get install libgmpxx4ldbl libgmp-dev libprocps4-dev libssl-dev
```

- 安装PlatON：

```bash
# 下载
$ wget https://download.platon.network/0.5/platon-ubuntu-amd64-0.5.0-with-mv.tar.gz

# 解压
$ tar -xvzf platon-ubuntu-amd64-0.5.0-with-mv.tar.gz
```

### PPA源安装

添加`PPA`源并安装`PlatON`软件包，安装方式如下：

1. 安装Platon基础包

```bash
# 添加PPA
$ sudo add-apt-repository ppa:platonnetwork/platon
$ sudo apt-get update

# 安装Platon基础包
$ sudo apt-get install platon
```

2. 安装带MPC及VC功能的Platon包

```bash
# 添加PPA
$ sudo add-apt-repository ppa:platonnetwork/platon
$ sudo apt-get update

# 安装带MPC及VC功能的Platon包
$ sudo apt-get install platon-all
```

安装完成后，可执行程序将安装到： `/usr/bin/`

### debian包安装

官方下载和安装：

1. 安装PlatON基础包

```bash
# 下载安装包 
$ wget https://download.platon.network/0.5/platon-ubuntu-amd64-0.5.0.deb

# 安装
$ sudo dpkg -i platon-ubuntu-amd64-0.5.0.deb
```

2. 安装带MPC及VC功能的PlatON包

> 注意：安装此包之前需要安装`MPC VM`和`VC`相关依赖，其中`MPC VM`依赖提供deb安装方式，并检查本机CPU架构是否在0.2及0.2以上！

安装`MPC VM`依赖

```bash
# 下载MPC lib安装包 
$ wget https://download.platon.network/0.5/platon-mpclib-ubuntu-amd64-0.5.0.deb

# 安装MPC lib
$ sudo dpkg -i platon-mpclib-ubuntu-amd64-0.5.0.deb

# 安装VC依赖
$ sudo apt-get install libboost-all-dev 
$ sudo apt-get install llvm-6.0-dev llvm-6.0 libclang-6.0-dev 
$ sudo apt-get install libgmpxx4ldbl libgmp-dev libprocps4-dev libssl-dev

# 下载PlatON安装包 
$ wget https://download.platon.network/0.5/platon-all-ubuntu-amd64-0.5.0.deb

# 安装
$ sudo dpkg -i platon-all-ubuntu-amd64-0.5.0.deb
```

安装完成后，可执行程序将安装到： `/usr/bin/`

### 源码编译安装

Ubuntu编译环境要求：

- 系统版本：`Ubuntu 16.04.1 及以上`，
- git：`2.19.1及以上`
- 编译器：`gcc(4.9.2+)`
- go语言开发包：`go(1.7+)`
- cmake:`3.0+` 

> ***注意：保证满足编译环境要求***

执行以下步骤：

#### 1. 获取源码

源码地址：<https://github.com/PlatONnetwork/PlatON-Go.git>，执行如下命令编译生成 `platon` 可执行文件:

```
$ git clone https://github.com/PlatONnetwork/PlatON-Go.git --recurive
```

#### 2. 编译

##### 编译无`MPC`及`VC`功能`PlatON`客户端

```bash
$ cd PlatON-Go
$ find ./build -name "*.sh" -exec chmod u+x {} \;
$ make all
```

##### 编译带`MPC`功能`PlatON`客户端

如需`platon`启用`MPC` 功能，请编译 `MPC VM`模块，并链接到PlatON中。步骤如下：

- 编译`MPC VM`

请参考[隐私合约虚拟机编译](https://github.com/PlatONnetwork/privacy-contract-vm#building--installing)。
假定编译目录为`~/home/path/to/mpcvm/build`，编译将输出`MPC VM`运行库到`~/home/path/to/mpcvm/build/lib`.

- 配置环境

将编译之后的`MPC VM`库`~/home/path/to/mpcvm/build/lib`路径添加到当前用户环境变量:

```bash
$ grep "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/mpcvm/build/lib" ~/.bashrc || echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:~/home/path/to/mpcvm/build/lib" >> ~/.bashrc

$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/home/path/to/mpcvm/build/lib
```

- 编译`platon`

```bash
$ cd PlatON-Go
$ find ./build -name "*.sh" -exec chmod u+x {} \;
$ make all-with-mpc
```

编译完成之后在`PlatON-Go/build/bin`目录下会生成`platon`、`ethkey`和`ctool`可执行文件，将此三个可执行文件拷贝到自己工作目录运行即可。

>**提示**：`MPC`计算功能是`PlatON`平台实现隐私计算提供的基础设施。更多MPC相关请[参考这里](zh-cn/development/[Chinese-Simplified]-%e9%9a%90%e7%a7%81%e5%90%88%e7%ba%a6%e5%bc%80%e5%8f%91%e6%8c%87%e5%8d%97)

##### 编译带`VC`功能`PlatON`客户端

- 编译`platon`

```bash
$ cd PlatON-Go
$ find ./build/ -name "*.sh" -exec chmod u+x {} \;
$ make all-with-vc
```

编译完成之后在`PlatON-Go/build/bin`目录下会生成`platon`、`ethkey`和`ctool`可执行文件，将此三个可执行文件拷贝到自己工作目录运行即可。

>**提示**：`VC`验证计算功能是`PlatON`平台实现隐私计算提供的基础设施。更多VC相关请[参考这里](zh-cn/development/[Chinese-Simplified]-可验证合约)

##### 编译带`MPC`及`VC`功能`PlatON`客户端

- 安装`VC`依赖

  请参考[编译带`VC`功能`PlatON`客户端](#编译带`VC`功能`PlatON`客户端)中安装`VC`依赖。

- 编译`MPC VM`

  请参考[编译带`MPC`功能`PlatON`客户端](#编译带`MPC`功能`PlatON`客户端)中编译`MPC VM`的方式安装MPC依赖。 

- 编译`platon`

```bash
$ cd PlatON-Go
$ find ./build/ -name "*.sh" -exec chmod u+x {} \;
$ make all-with-mv
```

编译完成之后在`PlatON-Go/build/bin`目录下会生成`platon`、`ethkey`和`ctool`可执行文件，将此三个可执行文件拷贝到自己工作目录运行即可。

## Windows安装

Window环境支持以下三种安装方式：

- 官方二进制包
- Chocolatey安装
- 源码

### 官方二进制下载安装

Windows各版本的`PlatON`二进制文件下载地址如下：

- Platon基础包：<https://download.platon.network/0.5/platon-windows-x86_64-0.5.0.zip>
- 带MPC及VC功能的Platon包：<https://download.platon.network/0.5/platon-all-windows-x86_64-0.5.0.zip>

下载后无需安装，直接解压即可使用，如需MPC功能则需要安装相关依赖，安装方式如下：

 windows版本`MPC VM`依赖下载地址：<https://download.platon.network/0.5/platon-mpclib-windows-x86_x64-0.5.0.zip>，下载之后解压文件，并配置环境变量，若解压后路径为：D:/platon/mpclib/，则添加的环境变量为:
 
 ```
 LD_LIBRARY_PATH = D:/platon/mpclib/
 ```

解压内容如下：

- `platon` PlatON客户端
- `ethkey` 公私钥对生成工具
- `ctool`  合约发布调用工具

### Chocolatey安装

1. 安装PlatON基础包

我们使用`chocolatey`包管理器来安装所需的构建工具。如果你还没有`chocolatey`，可以按照<https://chocolatey.org>上的说明进行安装。

用管理员身份启动`PowerShell`,然后使用`choco`命令安装`platonnetwork`：

```
> choco install platonnetwork --version=0.5.0
```

`platon`，`ethkey`等将默认被安装到`C:\ProgramData\chocolatey\bin`目录。

2. 安装带MPC及VC功能PlatON包

用管理员身份启动`PowerShell`,然后使用`choco`命令安装`platonnetwork-all`：

```
> choco install platonnetwork-all --version=0.5.0
```

`platon`，`ethkey`等将默认被安装到`C:\ProgramData\chocolatey\bin`目录。


### 源码编译安装

Windows编译环境需要符合以下条件：

- git：`2.19.1以上`
- go语言开发包：`go(1.7+)`
- mingw：`mingw（V6.0.0）`

说明：可自行安装以上编译环境，在编译`Platon`源码之前请确保以上环境可正常运行。也可使用`chocolatey`安装，可参考以下方式：

#### 1. 安装编译环境

用管理员身份启动`PowerShell`，然后执行以下命令：

```
// 安装git
> choco install git
// 安装golang
> choco install golang
// 安装mingw
> choco install mingw
```

利用`chocolatey`包管理器安装的软件大部分有默认的安装路径，部分软件可能会有各种各样的路径，这取决于软件的发布者。安装这些包将修改Path环境变量。最后安装路径可查看PATH。安装完之后请确保已安装的Go版本为1.7（或更高版本）。

#### 2. 获取`PlatON`源码

在当前`%GOPATH%`目录下创建`src/github.com/PlatONnetwork/`和`bin`目录，在`PlatONnetwork`目录下克隆`PlatON-GO`的源码:

```
> git clone https://github.com/PlatONnetwork/PlatON-Go.git
```

#### 3. 编译

在源码目录`PlatON-GO`下执行不同编译命令可生成带有不同功能的可执行文件，如下：

1. 编译PlatON基础包

```
> go run build/ci.go install ./cmd/platon
```

2. 编译带MPC功能的PlatON包
```
> go run build/ci.go install -mpc on ./cmd/platon
```

3.编译带MPC及VC功能的PlatON包
```
> go run build/ci.go install -mv on ./cmd/platon
```

编译完成之后在`PlatON-Go/build/bin`目录下会生成`platon`、`ethkey`和`ctool`可执行文件，将此三个可执行文件拷贝到自己工作目录运行即可。

## Docker环境运行

#### docker安装

docker安装比较简单，可参考以下两种安装方式：

- 从[阿里云Docker镜像源](https://yq.aliyun.com/articles/110806)安装
- 从[Docker官方镜像源](https://docs.docker.com/install/linux/docker-ce/ubuntu/)安装

#### 运行容器

- 拉取platon镜像

```bash
$ sudo docker pull platonnetwork/platon:tag #例如: platonnetwork/platon:0.5.0
```

- 运行platon容器

执行以下命令在容器中启动platon节点，其默认得RPC端口为6789，P2P端口为16789：

```bash
$ sudo docker run -d platonnetwork/platon:tag
```

若想开启端口映射，可执行以下命令：

```bash
$ sudo docker run -d -e PLATONIP="192.168.120.20" -p 6789:6789  -p 16789:16789 --name platon platonnetwork/platon:0.5.0
```

其中，`PLATONIP`为本机服务器地址。

执行以下命令进入容器(进入容器后默认目录就在/opt/node下):

```bash
$ sudo docker exec -it container_name bash #container_name为容器id或者名字
```

执行一下命令进入platon的js终端:

```bash
# platon attach ipc:./data/platon.ipc
```

注意：若没有开通端口映射，只能在容器内进入以上命令进入js终端，开启端口映射之后可在本机通过以下命令进入js终端：

```bash
# platon attach http://PLATONIP:6789
```

在容器内启停platon服务，使用如下命令：

```bash
# supervisorctl start platon #启动platon
# supervisorctl stop platon #停止platon
# supervisorctl restart platon #重启platon
```

如果以上命令无效，请使用以下命令

```bash
# /etc/init.d/supervisor stop
# /etc/init.d/supervisor start
```

**说明：容器中platon的数据目录位于`/opt/node`目录下，系统默认只有一个钱包账户，密码为:123456**