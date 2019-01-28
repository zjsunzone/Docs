
> 基于可验证计算(VC)密码学算法，`PlatON`提出了一种区块链链下计算扩容方案，通过可验证计算，合约只需要在链下计算一次，所有节点可以快速验证计算的正确性。

这种方案中的智能合约称为可验证合约，可验证合约的编写和发布跟Wasm合约没有区别，最终也是编译成wasm执行。可验证合约的状态转换在链下由计算节点异步执行，计算完成后新的状态和状态转换证明提交到链上，全网节点可快速验证正确性并将新的状态更新到公共账本中。可验证合约可支持复杂、繁重的计算逻辑而不影响整条链的性能。


## 总体架构

<div align=left>
<img src="zh-cn/development/verifiable-contract/images/vc-structure.png" width = "890" height="720" />
</div>

**架构说明**：

- 计算节点

计算节点是`PlatON`独创性地引入到区块链生态中的一类节点。计算节点本身也是一个`PlatON`全节点，同时也提供算力，在链下执行可验证合约，并使用VC算法生成计算证明，供链上节点进行快速验证。

- User Code

用户使用 `C++` 编写的可验证合约源码文件，经过 vlang 编译器编译后生成中间 `VCC.cpp` 文件。

- vlang

`vlang` 编译器，用来把用户编写的可验证合约源码程序编译为中间 `VCC.cpp` 文件，这个中间 `VCC.cpp` 文件中包含 `VKEY`、`EKEY`和加密物件(`gadgets`)的创建过程。

- vccMain.cpp

该文件是可验证合约框架文件，用户无需修改，该文件会默认放置到 `WASM` 编译工具链中。

- WASM toolchain

智能合约编译工具链，用于将`vccMain.cpp`和中间 `VCC.cpp`文件编译成WASM字节码。

- WASM (可验证Contract)

可验证合约的`WASM`字节码。

- WASM VM

执行可验证合约的 WASM 虚拟机。

## 快速开始

    案例说明

### 环境准备

#### 安装PlatON节点

用户可以选择在本地搭建私有网络，也可以将本地`PlatON`节点连接到贝莱世界测试网络来构建运行环境。

##### 方式一：搭建私有网络

验证合约运行只需要一个节点即可。由于`CPU`架构的兼容问题，`PlatON`发布的二进制安装包不支持可验证合约，因此用户需要通过源码安装，另外目前可验证合约只支持 `Ubuntu` 环境，过程如下:

 - 编译

```
# 安装依赖
$ sudo apt-get install libboost-all-dev 
$ sudo apt-get install llvm-6.0-dev llvm-6.0 libclang-6.0-dev 
$ sudo apt-get install libgmpxx4ldbl libgmp-dev libprocps4-dev libssl-dev
# 下载源码
$ git clone --recurive https://github.com/PlatONnetwork/PlatON-Go.git 
# 编译
$ cd PlatON-Go
$ find ./build/ -name "*.sh" -exec chmod u+x {} \;
$ make all-with-vc
```

- 启动节点

启动节点的具体方式请参考文档[私有网络](zh-cn/basics/[Chinese-Simplified]-私有网络)，注意节点的启动命令修改为如下形式：

```
$ ./platon --identity platon --datadir ./data --port 16789 --rpcaddr 0.0.0.0 --rpcport 6789 --rpcapi db,eth,net,web3,admin,personal --rpc --nodiscover --debug --verbosity 4  --vc --vc.actor 0xd7398978d04565ccf44097106e1cb7e9148d8ec9 --wasmlog wasm.log 
```

合约相关参数说明如下： 

`--vc`  表示节点将开启 `VC` 功能

`--vc.actor`  用来指定账户将计算结果和证明发布上链

`--wasmlog wasm.log`  指定该参数则执行VC计算的日志会输出到`wasm.log`，不指定则该日志会输出到节点日志。

**提示：**

> 计算节点启动后，网络中所有开启可验证功能的节点会自动监听并计算网络上所有可验证合约交易，第一个计算完成后的节点会将计算结果和证明提交到链上，此节点指定的 `vc.actor` 账户会扣除交易手续费，但同时会获得一笔计算费。

**警告：**

> 目前的实现方案中，命令行参数 `--vc.actor` 指定的账户需要解锁，存在一定的安全风险，后续会进一步优化。

##### 方式二： 连接测试网络

贝莱世界测试网络现已开放，可按照[测试网络连接指南](#zh-cn/basics/[Chinese-Simplified]%20快速指南#连接到贝莱世界测试网络)将计算节点连接到测试网络。

与私有网络一样，启动节点时也需要增加命令行参数：`--vc` 和 `--vc.actor`

#### 账户准备

节点启动之后需要解锁 `--vc.actor` 账户，作为计算节点参与可验证合约计算：

```
$ ./platon attach http://localhost:6789
> personal.unlockAccount("0xd7398978d04565ccf44097106e1cb7e9148d8ec9","password", 2592000)
> true
```

账户 `0xd7398978d04565ccf44097106e1cb7e9148d8ec9` 是用来参与验证计算的账户，现创建并解锁另一账户用来后续发起计算请求：

```
> personal.newAccount("1111")
"0xdb20c32e018b6ba9626936323d7c0744eaca5ef2"
> personal.unlockAccount("0xdb20c32e018b6ba9626936323d7c0744eaca5ef2","1111", 2592000)
> true
```

**注意**：需要确保以上两个账户内`Energon`余额充足，单节点直接转账即可。若是以连接测试网络的方式测试，则以上两个账户均需要在官网申请足够的测试 `Energon` 才能参与计算，申请地址：<https://developer.platon.network/#/energon?lang=zh>

### 编写VC合约

一个简单的可验证合约如：`Add.cpp`

```bash
    int vc_main(int a, int b) {
        return a+b;
    }
```

注意：可验证合约可用 C/C++ 语法编写，但是有以下限制：

1. 主函数的函数名必须为vc_main
2. 主函数参数个数不限，但是参数类型目前只支持基本的数值类型（如：char,short,int,long）
2. vc_main函数原型不能出现指针和引用类型
3. 循环/递归次数必须在编译时确定
4. 数组寻址下标必须是常数
5. 不能使用IO操作
6. 不能使用动态内存分配
7. 不能使用浮点类型

在编写前请注意阅读并理解以上限制条件。

### 编译VC合约

#### 1. 预编译

下载`VC-Compile`源码： <https://github.com/PlatONnetwork/vc-compiler.git>，将编译后的 `vlang` 工具拷贝到自己的工作目录。`vlang`详细用法用法如下:

```
vlang [clang args] [options] <inputs>
clang args:     clang/gcc 支持的选项
    --help           打印选项
    -args=arg        设置调试参数
    -ir file         llvm IR输出到 file (默认不输出)
    -o  file         目标CPP文件输出到 file (默认输出名称为vcc.cpp)
options:        vlang 的附加选项
inputs:         输入文件（可以输入多个文件，会被链接起来）
```

预编译是把可验证合约源码文件(Add.cpp)使用 `vlang` 编译器编译为中间 `CPP` 文件，如：`vcc.cpp`

```bash
$ ./vlang Add.cpp -o vcc.cpp
```

编译后会在当前目录下生成`vcc.cpp`文件。


#### 2. 编译`vcc.cpp`

预编译生成的 `vcc.cpp` 需要编译为`WASM`字节码，编译环境请参考[配置WASM合约开发环境](zh-cn/development/wasm-contract/_配置Wasm合约开发环境#%E9%85%8D%E7%BD%AEwasm%E5%90%88%E7%BA%A6%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)，其编译过程和普通`WASM`合约不同，其编译方式如下：

- 创建Wasm合约工程

```shell
$ cd {pWASM}
$ ./script/autoproject.sh . demo vc
```


该 `demo` 工程会默认在 `user` 目录生成。此时新生成的工程目录结构为：

```text
user
├── CMakeLists.txt
├── demo
│   ├── CMakeLists.txt
│   └── demo.cpp
└── vccMain.cpp
```
**注意：**

>  `demo.cpp` 为可验证合约框架代码，此文件千万不能修改！文件`vccMain.cpp`作为一个模板文件，不能修改也不能删除！

- 把预编译生成的 `vcc.cpp` 放置到user/demo/ 目录下，修改该目录下的`CMakeLists.txt` 文件，在`SOURCE_FILES`中添加 `vcc.cpp` 和 `demo.cpp`，如下：

```
add_wast_executable(TARGET demo 
     #Multiple cpp files use SOURCE_FILES, and the name of TARGET is the same as the cpp file name with PLATON_ABI 
     SOURCE_FILES demo.cpp vcc.cpp
     INCLUDE_FOLDERS ${STANDARD_INCLUDE_FOLDERS} 
     LIBRARIES  ${libplaton} ${libc++} ${libc} 
     DESTINATION_FOLDER ${CMAKE_CURRENT_BINARY_DIR})
```

- 编译

```bash
$ cd {pWASM}/build/
$ make
```

执行完成后，在`{pWASM}/build/user/demo`目录结构如下：

```
├── demo
    ├── CMakeFiles
    ├── cmake_install.cmake
    ├── demo.bc
    ├── demo.cpp.abi.json
    ├── demo.cpp.bc
    ├── demo.s
    ├── demo.wasm
    ├── demo.wast
    ├── Makefile
    └── vcc.cpp.bc
```

### 发布合约

将上一步骤编译 `vcc.cpp` 文件生成的 `demo.wast`， `demo.cpp.abi.json` 发布上链，发布方式请参考[Wasm合约发布](zh-cn/development/wasm-contract/_配置Wasm合约开发环境#发布合约)。

合约发布成功后，返回合约地址如下：

```
contract address: 0x139784f914759639aa750b9a2da4dcd652d71986
```

> 提示: 合约发布之后在控制台使用 `eth.getCode("0x139784f914759639aa750b9a2da4dcd652d71986")`，查看是否能获取code，如果返回"0x"，则表示发布失败，后续步骤将执行不了！此时可以尝试上调 `config.json` 文件中 gas 值

### 发起验证计算

目前提供以下两种方式发起验证计算：

- 方式一：使用工具 `vc_tool`

工具`vc_tool`在`PWASM`包下有提供，发起计算命令如下。

```bash
$./vc_tool -from=0xdb20c32e018b6ba9626936323d7c0744eaca5ef2 -to=0x139784f914759639aa750b9a2da4dcd652d71986 -gas=0x166709 -gasPrice=0x8250de00 -value=0x4F2F63531C4A8 -func=compute -parms=123#100
```

参数说明如下：

```
- from ：发起验证计算请求的钱包地址，为验证测试结果，该地址最好不要跟计算节点的收款账户相同。此处使用的是账户准备时生成的钱包地址
- to   ：合约地址，发布合约步骤中返回的合约地址
- value：为本次计算提供的费用
- func ：合约方法，目前固定为compute
- parms：VC合约计算方法中的参数，多个参数则以`#`进行分割（该例子中为Add.cpp）
```

返回如下信息则表示正常：

```
from:  0xdb20c32e018b6ba9626936323d7c0744eaca5ef2
to  0x139784f914759639aa750b9a2da4dcd652d71986
map[params:[map[to:0x139784f914759639aa750b9a2da4dcd652d71986 gas:0x166709 gasPrice:0x8250de00 value:0x4F2F63531C4A8 data:0xd688000000000000000687636f6d707574658431303030 from:0xdb20c32e018b6ba9626936323d7c0744eaca5ef2]] id:1 jsonrpc:2.0 method:eth_sendTransaction]
{"jsonrpc":"2.0","id":1,"result":"0x54a699528c7dcef05fdf2b0a8ce9bff4c956a7fd48e51bde773a19be5dd80217"}
```

- 方式二：Java调用

 1. 创建普通的Java Maven工程，修改`pom.xml`,添加maven仓库地址

```xml
<repositories>
    <repository>
        <id>juzixmaven</id>
        <name>juzix maven</name>
        <url>https://sdk.platon.network/nexus/content/groups/public/</url>
    </repository>
</repositories>
```

2. 引入`vc-data-sdk`依赖

```xml
<dependency>
  <groupId>net.platon.vc</groupId>
  <artifactId>vc-data-sdk</artifactId>
  <version>1.0.0-RELEASE</version>
</dependency>
```

* 编写代码

新增应用主类用来编写程序入口，内容如下：

```java
public class Client {
    public static void main(String[] args) {
        /* 初始化客户端
         * 参数1为节点地址
         * 参数2为发起请求的账户私钥文件路径
         * 参数3为发起请求的账户解锁密码
         */
        ProxyClient proxyClient = new ProxyClient(
                "http://192.168.112.71:6789",
                "D:\\resource\\platon\\temp\\db20c32e018b6ba9626936323d7c0744eaca5ef2",
                "1111");
          
        /* 设置交易参数
         * 参数3为发起请求提供的计算费
         * 参数4为合约地址
         */
        proxyClient.setTransactionParams(null, null, BigInteger.valueOf(207499L),
                "0x139784f914759639aa750b9a2da4dcd652d71986");
                
        //发起计算，返回交易hash
        String hash = proxyClient.startCalc("compute", "1000#20", 1);
        System.out.println(hash);
    }
}
```

通过以上两种方式发起计算成功之后，在`wasm.log`日志中会出现以下内容：

```
init vccMain
gen vc task id :  7d57bf3bad8a6362218854fdf65edbd5455f0d81de7e4dc13970eae046055da6
cost value: 1393039175173288
save_cost_value:  VC_CV7d57bf3bad8a6362218854fdf65edbd5455f0d81de7e4dc13970eae046055da6  -  1393039175173288
```
由于需要经过20个区块之后才会执行计算，所以等待一段时间之后会继续打印以下信息:

```
init vccMain
gen vc task id :  7d57bf3bad8a6362218854fdf65edbd5455f0d81de7e4dc13970eae046055da6
cost value: 1393039175173288
save_cost_value:  VC_CV7d57bf3bad8a6362218854fdf65edbd5455f0d81de7e4dc13970eae046055da6  -  1393039175173288
get user input: 1000#20
Gen proof & result success...
Gen proof success res: 0 21799650504762465282144718950080893131863466887298723316447067322305804200260 1 0 21350921935594571139830291726833717650903173730283803760219831755018046025900 1
0 7245508092219627051320200861282160007229615091802556391344724239190025255206 20416339134969601553435214934927383600721349863109021002226987949196053812857 1 0 8431162032868089988594581170979886809847772404759863243011326547875059572526 1
0 20194036651331952170090120231283264061580421887557029554367607852982696768475 0 0 19183644847953964407999739086741219314155422913858600999029833731252972739533 0
0 4618040147889319282783728832715633290970125453626917143991023738496049083633 0
0 8470982052219203871236879467230502757025995998621521234806210887932803921696 1
#3fc
proof is  0 21799650504762465282144718950080893131863466887298723316447067322305804200260 1 0 21350921935594571139830291726833717650903173730283803760219831755018046025900 1
0 7245508092219627051320200861282160007229615091802556391344724239190025255206 20416339134969601553435214934927383600721349863109021002226987949196053812857 1 0 8431162032868089988594581170979886809847772404759863243011326547875059572526 1
0 20194036651331952170090120231283264061580421887557029554367607852982696768475 0 0 19183644847953964407999739086741219314155422913858600999029833731252972739533 0
0 4618040147889319282783728832715633290970125453626917143991023738496049083633 0
0 8470982052219203871236879467230502757025995998621521234806210887932803921696 1

result is  3fc
input is  1000#20
verify pass.
set result taskid : VC_RES7d57bf3bad8a6362218854fdf65edbd5455f0d81de7e4dc13970eae046055da6
get_cost_value:  1393039175173288
get_cost_value to: 1393039175173288
transfer to: d7398978d04565ccf44097106e1cb7e9148d8ec9  value: 1393039175173288
set_result success. 0
```

出现以上日志表示计算成功，并且计算结果已经上链。

### 获取计算结果

目前在工具中不提供查询结果的方法，只能通过 `sdk` 获取，获取方式如下：

```java
public class Client {
    public static void main(String[] args) {
        //初始化客户端
        ProxyClient proxyClient = new ProxyClient(
                "http://192.168.112.71:6789",
                "D:\\resource\\platon\\temp\\db20c32e018b6ba9626936323d7c0744eaca5ef2",
                "1111");
          
        //设置交易参数,这里主要设置合约地址，其他参数不需要
        proxyClient.setTransactionParams(null, null, null,
                "0x139784f914759639aa750b9a2da4dcd652d71986");
                
        //上一步骤发起计算之后返回的hash
        String hash = "0x9883166edf14600a8d0b6c9fe9034993ff1adc75184fd2ec0fd7bc335c8d7cba";
        String taskId = proxyClient.getTaskId(hash);
        
        //通过tashkId获取结果
        String result = proxyClient.getResultByTaskId(taskId, 100000);
        System.out.println(result);
    }
}
```
运行后控制台输出：

```
1020
```

>提示：计算完成之后，用户可自行查询账户余额增减。

## 可验证合约SDK

1. 客户端构造函数

```
public ProxyClient(String url, String walletPath, String walletPass)
```

参数说明如下：

```
- url：节点的`RPC`端口
- walletPath：发起计算的钱包地址
- walletPass：对应钱包的解锁密码
```

2. 设置交易参数

```
public void setTransactionParams(BigInteger gasPrice, BigInteger gasLimit, BigInteger value, String contractAddress)
```

参数说明如下：

```
- gasPrice：发送web3j请求gasPrice,默认为22000000000L
- gas     ：发送web3j请求的gas,默认为4300000
- value   ：发送web3j请求的value值,在发起计算的不传递默认为4300000，查询结果该默认值为0
- contractAddress：发布合约之后的地址
```

3. 开启vc计算

```
public String startCalc(String method, String params, int retry)
```

参数说明如下：

```
- method：发送方法名,null默认为compute
- params：发送交易请求的参数，多个参数以#分割
- retry ：发送交易的重试次数
- 返回  ：结果上链交易hash
```

4. 根据交易hash值查询任务对应 taskId

```
public String getTaskId(String transactionHash)
```

参数说明如下：

- hash：计算请求返回的hash值
- 返回：交易hash

5. 根绝taskId查询任务结果

***注意：因为要间隔20个块进行vc计算，所以结果一般要等20秒之后才能查到计算结果***

```
public String getResultByTaskId(String taskId, final long timeout) 
```

参数说明如下：

- taskId  ：计算任务的taskId
- timeout ：查询超时时间
