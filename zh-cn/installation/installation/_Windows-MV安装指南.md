# Windows安装

Window环境支持以下三种安装方式：

- [Chocolatey安装](#Chocolatey安装)
- [二进制包](#二进制包安装)
- [源码](#源码编译安装)

## Chocolatey安装

用管理员身份启动`PowerShell`,然后使用`choco`命令安装`platonnetwork-all`：

```
> choco install platonnetwork-all --version=0.5.0
```

`platon`，`ethkey`等将默认被安装到`C:\ProgramData\chocolatey\bin`目录，并加入`PATH`环境变量。


## 二进制包安装
1. 创建`PlatON`工作目录`%PLATON%`
2. 从[这里](https://download.platon.network/0.5/platon-mpclib-windows-x86_x64-0.5.0.zip)下载MPC依赖包，并解压到`%PLATON%\mpclib`
3. 从[这里](https://download.platon.network/0.5/platon-windows-x86_64-0.5.0-with-mv.zip)下载带MPC及VC功能的`PlatON`二进制包，并解压到工作目录`%PLATON%`
   解压内容如下：

   - `platon` PlatON客户端
   - `ethkey` 公私钥对生成工具
   - `ctool`  合约发布调用工具


## 源码编译安装

**Windows编译环境需要符合以下条件：**

- git：`2.19.1以上`
- go语言开发包：`go(1.7+)`
- mingw：`mingw（V6.0.0）`
- cmake: `3.0+`

可自行安装以上编译环境，在编译`Platon`源码之前请确保以上环境可正常运行。也可使用`chocolatey`安装，可参考以下方式：

### 1. 安装编译环境

用管理员身份启动`PowerShell`，然后执行以下命令：

```
// 安装git
> choco install git
// 安装golang
> choco install golang
// 安装mingw
> choco install mingw
// 安装cmake
> choco install cmake
```

利用`chocolatey`包管理器安装的软件大部分有默认的安装路径，部分软件可能会有各种各样的路径，这取决于软件的发布者。安装这些包将修改Path环境变量。最后安装路径可查看PATH。安装完之后请确保已安装的Go版本为1.7（或更高版本）。

### 2. 获取`PlatON`源码

在当前`%GOPATH%`目录下创建`src/github.com/PlatONnetwork/`和`bin`目录，在`PlatONnetwork`目录下克隆`PlatON-GO`的源码:

```
> cd src/github.com/PlatONnetwork/
> git clone https://github.com/PlatONnetwork/PlatON-Go.git  --recursive
```

### 3. 编译

> **注意:以下编译命令均需在`Git-bash`环境运行， 且步骤1中编译环境都安装！**

在编译源码目录之前需要在源码目录`PlatON-GO`下执行以下脚本编译所需依赖库：

```
> cd PlatON-GO
> ./build/build_deps.sh
```

在源码目录`PlatON-GO`下执行以下编译命令生成可执行文件`platon`、`ethkey`和`ctool`：

```
> cd PlatON-GO
> go run build/ci.go install -mv on ./cmd/platon
> go run build/ci.go install ./cmd/ethkey
> go run build/ci.go install ./cmd/ctool
```

### 4. 可执行文件安装

编译完成之后在`PlatON-Go/build/bin`目录下会生成`platon`、`ethkey`和`ctool`的执行文件，将此三个可执行文件拷贝到工作目录`%PLATON%`。

### 5. 依赖包安装
从[这里](https://download.platon.network/0.5/platon-mpclib-windows-x86_x64-0.5.0.zip)下载MPC依赖包，并解压到目录`%PLATON%\mpclib`
