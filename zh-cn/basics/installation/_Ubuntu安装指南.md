
## Ubuntu安装

### 环境说明：
 
 本文档提供的安装方式支持Ubuntu系统版本：`Ubuntu 16.04 及以上`

### 安装方式： 

- [二进制包](#二进制包安装)
- [PPA源](#PPA源安装)
- [Debian](#Debian安装)
- [源码](#源码编译安装)

> **注意：由于目前官方提供的二进制包、PPA源和Debian包只支持16.04版本，在非16.04版本的环境安装`PlatON`只能通过编译源码的方式安装。**

#### 二进制包安装

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

#### PPA源安装

添加`PPA`源，然后安装`PlatON`客户端，如下：

```bash
# 添加PPA
$ sudo add-apt-repository ppa:platonnetwork/platon
$ sudo apt-get update

# 安装PlatON
$ sudo apt-get install platon
```

安装完成后，可执行程序将安装到： `/usr/bin/`。

#### Debian安装

```bash
# 下载安装包 
$ wget https://download.platon.network/0.5/platon-ubuntu-amd64-0.5.0.deb

# 安装
$ sudo dpkg -i platon-ubuntu-amd64-0.5.0.deb
```

安装完成后，可执行程序将安装到： `/usr/bin/`。

#### 源码编译安装

Ubuntu编译环境要求：

- 系统版本：`Ubuntu 16.04.1 及以上`，
- git：`2.19.1及以上`
- 编译器：`gcc(4.9.2+)`
- go语言开发包：`go(1.7+)`
- cmake:`3.0+` 


#### 1. 获取源码

```
$ git clone https://github.com/PlatONnetwork/PlatON-Go.git --recursive
```

#### 2. 编译

```bash
$ cd PlatON-Go
$ find ./build -name "*.sh" -exec chmod u+x {} \;
$ make all
```

编译完成之后在`PlatON-Go/build/bin`目录下会生成`platon`等一系列可执行文件，可根据需求将可执行文件拷贝到自己工作目录运行即可。

