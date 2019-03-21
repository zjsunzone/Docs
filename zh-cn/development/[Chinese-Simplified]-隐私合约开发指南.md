# 隐私合约开发指南

## 简介

> [PlatON](zh-cn/basics/[Chinese-Simplified]-快速指南) 平台作为下一代`Trustless`安全数据计算架构，基于安全多方计算(`MPC`)对数据隐私保护特性，使用隐私合约实现隐私计算业务逻辑，为隐私计算提供了基础设施。

隐私合约使用C++语言编写，通过编译、部署等步骤发布到`PlatON链上`，并借助`PlatON`线上数据节点和线下计算节点组成的复合网络，接收隐私计算客户端（需求方）发起的隐私计算请求，提供隐私计算服务。

### 隐私计算架构

<div align=left>
<img src="privacy-contract/images/mpc_structure.png" width = "650" height="523"/>  
</div>



架构图说明：

- PlatON Network

  `PlatON` 区块链节点组成的网络，包含计算节点和其他非计算节点。

- Privacy Contract

   隐私合约，部署在区块链上，实现隐私计算业务逻辑。

- Computation Node

   计算节点，作为PlatON网络节点，包含隐私算法执行虚拟机，提供隐私计算运行时服务。

- Data Node

  数据节点，作为隐私数据提供者，协助计算节点完成隐私计算服务。

- Privacy Computation Client

  隐私计算客户端，作为隐私计算请求发起方，支付计算费用，并能够解密隐私计算结果。

  

## 快速开始

现在以MPC领域的经典案例 [姚氏百万富翁问题](https://en.wikipedia.org/wiki/Yao%27s_Millionaires%27_Problem)介绍隐私合约开发。

**案例表述**：

两个百万富翁`Alice`和`Bob`想知道他们两个谁更富有，但他们都不想让对方知道自己财富的任何信息。

**解决方案**：

使用`MPC`计算，两个富翁可以在不知道对方具体财产的情况下，与对方进行比较，并计算出谁最富有。

### 搭建环境

- 搭建环境参考： [搭建隐私开发环境](zh-cn/development/privacy-contract/_搭建隐私计算环境)。

- 配置工作目录：

  创建工作目录`YaoProblem`，隐私开发环境编译工具安装到本目录，工作区目录树类似如下：

  ```bash
  .
  ├── bin
  |   ├── protoc                                    # protobuf编译工具
  |   ├── plang                                     # plang隐私合约编译器
  ├── config										  # 配置目录
  ├── include                                		  #emp-tool库头文件
  |       ├── integer.h                         	  # 隐私计算对象头文件
  |       └── ...
  ├── lib                                            # clang头文件夹
  ├── config.json                                    # 配置文件
  ├── SimpleAddProto.cpp                             # 范例隐私合约
  └── ...
  ```

  

### 创建钱包

 对隐私计算参与方双方，隐私计算请求者，分别创建3个钱包文件和对应账户：

**Windows命令行**

```bash
> mkdir data
> platon.exe --datadir .\data account new
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat passphrase:
Address: {0x9a568e649c3a9d43b7f565ff2c835a24934ba447}Copy to clipboardErrorCopied
```

**Linux命令行**：

```bash
$ mkdir -p data
$ ./platon --datadir ./data account new
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat passphrase:
Address: {0x9a568e649c3a9d43b7f565ff2c835a24934ba447}Copy to clipboardErrorCopied
```

以上用户工作平台（Linux或Windows）执行3次创建钱包操作，例如本次示例输出：：

> ` 0x9a568e649c3a9d43b7f565ff2c835a24934ba447`

> `0xce3a4aa58432065c4c5fae85106aee4aef77a115`

> `0x9a568e649c3a9d43b7f565ff2c835a24934ba447`

由于后续计算需要消耗`energon`, 因此需要对着3个钱包地址分别充值。

- 测试网络环境
  在[PlatON官网](https://developer.platon.network/#/energon?lang=zh)申请测试`energon`。
- 私有网络环境
  从私有网络共识节点`coinbase账号`转账获取`energon`。

### 编写隐私合约

编写一个解决姚氏百万富翁问题的隐私合约 `YaoMillionairesProblem.cpp` 。代码如下：

```c++
#include <iostream>
#include "integer.h" // 使用MPC库头文件

/*
 * Yao's millionaires' problem, show who is richer.
 * @param money1, Alice参与方
 * @param money2, Bob参与方
 * @return true if Alice >= Bob else false.
 */
bool YaoMillionairesProblem(int money1, int money2) {
    std::cout << __FUNCTION__ 
        << " Alice: " << money1 
        << " Bob: " << money2 
        << std::endl;
	
    emp::Integer alice_money(money1, emp::ALICE);  // 设置Alice金额
    emp::Integer bob_money(money2, emp::BOB);  // 设置Bob金额

    int ret = (alice_money - bob_money).reveal();  // 执行比较大小，获取比较结果
    std::cout << __FUNCTION__ << " result(=Alice-Bob): " << ret << std::endl;

    return ret >= 0;
}
```



### 编译隐私合约

#### 更新配置文件

`config.json`用于配置计算中参与者信息、结算规则、接口定价等，用户需要根据自己的计算要求更新配置。本范例配置如下：

```json
{
    "invitor": "0x9a568e649c3a9d43b7f565ff2c835a24934ba447",
    "parties": [
        "0x9a568e649c3a9d43b7f565ff2c835a24934ba447",
        "0xce3a4aa58432065c4c5fae85106aee4aef77a115"
    ],
    "method-price": {
        "YaoMillionairesProblem": 200000
    },
    "profit-rules": {
        "0x9a568e649c3a9d43b7f565ff2c835a24934ba447": 1,
        "0xce3a4aa58432065c4c5fae85106aee4aef77a115": 2
    },
    "urls": {
        "0x9a568e649c3a9d43b7f565ff2c835a24934ba447": "DirectNodeServer:default -h 127.0.0.1 -p 10001",
        "0xce3a4aa58432065c4c5fae85106aee4aef77a115": "DirectNodeServer:default -h 127.0.0.1 -p 10002"
    }
}
```

配置文件说明：

- `invitor`    数据节点中计算邀请方的钱包地址。

* `parties`    两个数据节点对应的钱包地址列表。
* `method-price`    MPC算法函数定价（或收费），示例中将函数`YaoMillionairesProblem`定价为 **200000**。
* `profit-rules`    指定计算结束后的收费分配规则，按照比例分配，示例中分配比例为1：2。
* `urls`    指定计算参与方MPC服务地址信息。每个元素的key 为数据提供方钱包地址，value值为MPC服务地址信息(ICE规范地址，其中 `DirectNodeServer` 固定，`-h` 和 `-p` 分别为节点地址和 MPC 服务端口)。

#### 编译

编译命令如下，

Linux平台：

```bash
$ ./bin/plang ./YaoMillionairesProblem.cpp -config ./config.json -I ./include -mpcc YaoMillionairesContract.cpp
```

Windows平台：

```bash
> .\bin\plang YaoMillionairesProblem.cpp -config config.json -I .\include -mpcc YaoMillionairesContract.cpp
```

终端输出：

```
digest:
 * IR NAME: MPCYaoMillionairesProblem
 * IR HASH: be4bfbbc708c5622e0ff6f274020d654
 * <p>
 * IR FUNC HASH(MD5)                 IR FUNC NAME            IR FUNC PROT
 * 0588f14217b11e0f77e50d03a88ba866  YaoMillionairesProblem  YaoMillionairesProblem(int,int)
```

编译将生成的隐私合约、Java隐私合约代理类，文件目录树如下:

```
├── code
│   └── java
│       ├── MPCYaoMillionairesProblem.java              # 给数据提供方使用
│       ├── MPCYaoMillionairesProblem-README.TXT
│       ├── ProxyYaoMillionairesProblem.java            # 给计算发起方使用
│       └── ProxyYaoMillionairesProblem-README.TXT
└── YaoMillionairesContract.cpp                         # 生成WASM合约，最终上链的合约文件
```

更多 `plang` 编译器使用详情，参考 [Usage](https://github.com/PlatONnetwork/privacy-contract-compiler#Usage)。

**注意：配置文件或隐私合约代码的任何更新，都需要重新编译才能生效。**

### 隐私合约发布

隐私合约运行在隐私计算网络，需要发布到 `PlatON` 网络，使用前面步骤生成的 [Wasm合约](zh-cn/development/[Chinese-Simplified]-Wasm%e5%90%88%e7%ba%a6%e5%bc%80%e5%8f%91%e6%8c%87%e5%8d%97#Wasm%e5%90%88%e7%ba%a6%e7%ae%80%e8%bf%b0) 文件，可以通过交易的方式，部署到`PlatON`网络。

下面演示在Ubuntu 上发布Wasm 合约，以完成隐私合约发布流程。更多 Wasm合约开发信息请参考 [Wasm合约开发指南](zh-cn/development/[Chinese-Simplified]-Wasm%e5%90%88%e7%ba%a6%e5%bc%80%e5%8f%91%e6%8c%87%e5%8d%97#%e7%bc%96%e8%af%91%e5%90%88%e7%ba%a6)。

1. 下载 Ubuntu 版本 [Wasm合约开发包](https://download.platon.network/0.4/pwasm-linux-amd64-0.4.0.tar.gz) 并解压到 ${pWasm} 。若已下载，跳至下一步。

2. 进入 ${pWasm} 目录，执行以下命令生成Wasm合约项目 `YaoProblem`。

```bash
$ cd {pWasm}
$ ./script/autoproject.sh . YaoProblem
```

3. 拷贝隐私合约编译后的文件 `YaoMillionairesContract.cpp` 内容到 `{pWasm}/user/YaoProblem/YaoProblem.cpp`，然后进入 `{pWasm}/build` 编译。

```bash
$ cp YaoMillionairesContract.cpp ${pWasm}/user/YaoProblem/YaoProblem.cpp
$ cd {pWasm}/build
$ make
```

4. 创建 `config.json` 配置文件，配置当前合约发布的信息：

```bash
{
  "url":"http://127.0.0.1:6789",
  "gas":"0x99999788888",
  "gasPrice":"0x333330",
  "from":"0x9a568e649c3a9d43b7f565ff2c835a24934ba447"
}
```

>  **说明：发布合约可以是由任意账户发起。**


5. 连接 `http://127.0.0.1:6789` 节点，解锁 `0x9a568e649c3a9d43b7f565ff2c835a24934ba447` 账户。

```bash
$ platon 
> personal.unlockAccount("your-account")
Unlock account 0x9a568e649c3a9d43b7f565ff2c835a24934ba447
Passphrase:
true
```

6. 发布 Wasm 合约。

```bash
$ chmod u+x ctool
$ ./ctool deploy --abi ./YaoProblem.cpp.abi.json --code ./YaoProblem.wasm --config ./config.json
```

假定合约发布后的合约地址为：
> 0x43355c787c50b647c425f594b441d4bd75198888


### 数据节点服务实现

开发者需要使用隐私数据服务开发包(`mpc-data-sdk`)构建隐私数据服务，更多接口详情参考[隐私数据服务开发包](privacy-contract-develop-manual#隐私数据服务开发包)。

#### 构建工程

> `mpc-data-sdk` 只提供 `JAVA` 版本，因此读者需具备一定的 JAVA 编程基础。

使用`Intellij`创建两个Maven工程，分别对应Alice数据节点服务程序和Bob数据节点服务程序。进行如下操作：

* 创建 JAVA 普通 Maven 工程，修改`pom.xml`，添加maven仓库地址

```xml
<repositories>
   <repository>
        <id>juzixmaven</id>
        <name>juzix maven</name>
        <url>https://sdk.platon.network/nexus/content/groups/public/</url>
    </repository>
</repositories>
```

* 引入`mpc-data-sdk`依赖

```xml
<dependency>
    <groupId>net.platon.mpc</groupId>
    <artifactId>mpc-data-sdk</artifactId>
    <version>1.0</version>
</dependency>
```

* 编码实现

新建 `net.platon.vm.mpc` 包，添加 `plang` 编译隐私合约生成的 JAVA 代码 `MPCYaoMillionairesProblem.java` 到`net.platon.vm.mpc` 包(***注意***：**不要修改包名！**)，然后分别对应Alice数据节点服务程序和Bob数据节点服务程序实现`inputImpl`方法。

#### 接口实现

**Alice数据节点服务程序:**

实现 `MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_01` 类中的`inputImpl`方法，代码实现如下：

```java
/**
 * YaoMillionairesProblem(int,int)
 */
final class MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_01 extends mpc_f_0588f14217b11e0f77e50d03a88ba866_01 {
    public byte[] inputImpl(final InputRequestPara para) {
        int ret_value = 0;
        // TODO: assemble data

        ret_value = 1000000; // For Example Alice Input 1 million

        return Data.Int32(ret_value);
    }
}
```

**Bob数据节点服务程序:**

实现 `MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_02` 类中的方法，代码实现如下：

```java
/**
 * YaoMillionairesProblem(int,int)
 */
final class MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_02 extends mpc_f_0588f14217b11e0f77e50d03a88ba866_02 {
    public byte[] inputImpl(final InputRequestPara para) {
        int ret_value = 0;
        // TODO: assemble data

        ret_value = 2000000; // For Example Bob Input 2 million

        return Data.Int32(ret_value);
    }
}
```

类名格式说明：

`MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_01` 

	- MPCYaoMillionairesProblem        MPC前缀 + 隐私合约中算法函数名
	- YaoMillionairesProblem           方法名
	- int_int                          两个输入参数的类型
	- 01                               数据提供方编号，Alice为01，Bob为02

`mpc_f_0588f14217b11e0f77e50d03a88ba866_01`

	- mpc_f                             特定前缀
	- 0588f14217b11e0f77e50d03a88ba866  方法 YaoMillionairesProblem函数签名的MD5值
	- 01                                数据提供方编号，Alice为01，Bob为02

> **注意：数据节点程序务必实现代理类的 `InputImpl` 方法**

#### 实现程序入口main

两个工程都增加数据节点服务程序入口类，如下：

```java
public class Client {
    public static void main(String[] args) {
        ConfigInfo cfgInfo = new ConfigInfo(args);
        // 可以在此处对cfgInfo中的参数进行硬编码
        // cfgInfo.walletPath=""
        // cfgInfo.walletPass=""
        // cfgInfo.iceCfgFile=""
        AppClient app = new AppClient(cfgInfo);
        app.start(args);
    }
}
```

程序启动需要传入3个参数，参数说明：

| 参数       | 说明                         | 可空 |
| :--------- | :--------------------------- | :--- |
| walletPath | 隐私数据提供方钱包文件路径 | 否   |
| walletPass | 账户密码                     | 否   |
| iceCfgFile | ICE配置文件                  | 否   |

#### 打包数据节点程序

将两个工程项目分别打包，在项目`pom.xml`所在路径下，执行：

```bash
$ mvn build
```

两个数据节点服务程序分别输出： `mpc-data-sdk-client1-1.0-SNAPSHOT.jar`， `mpc-data-sdk-client2-1.0-SNAPSHOT.jar`。

#### 配置数据节点程序

本例假定搭建为私有网络，且在同一台机器上实现。

`Alice` 数据节点创建配置文件 `cfg.client1.config` ，配置内容如下：

```config
TaskCallback.Proxy=tasksession:default -h 127.0.0.1 -p 10001
Callback.Client.Endpoints=default -h 127.0.0.1
```

`Bob` 数据节点创建配置文件`cfg.client2.config` ，配置内容如下：

```config
TaskCallback.Proxy=tasksession:default -h 127.0.0.1 -p 10002
Callback.Client.Endpoints=default -h 127.0.0.1
```

配置文件说明：

- `TaskCallback.Proxy` 的地址和端口与[启动带MPC计算功能](zh-cn/basics/[Chinese-Simplified]-%e7%a7%81%e6%9c%89%e7%bd%91%e7%bb%9c#%e4%b8%ba%e8%8a%82%e7%82%b9%e5%90%af%e7%94%a8MPC%e5%8a%9f%e8%83%bd)的 `PlatON` 节点时，参数 `--mpc.ice` 指定的 ICE 配置文件中配置的地址和端口保持一致（默认端口是8201）。
- `Callback.Client.Endpoints`为启动数据隐私服务所在的服务器地址。

#### 程序运行

运行Alice数据节点：

```bash
$ java -jar mpc-data-sdk-client1-1.0-SNAPSHOT.jar --walletPath=./config/9a568e649c3a9d43b7f565ff2c835a24934ba447 --walletPass=11111111 --iceCfgFile=./config/cfg.client1.config
```

运行Bob数据节点：

```bash
$ java -jar mpc-data-sdk-client2-1.0-SNAPSHOT.jar --walletPath=./config/ce3a4aa58432065c4c5fae85106aee4aef77a115 --walletPass=11111111 --iceCfgFile=./config/cfg.client2.config
```

数据节点程序启动后，将注册到计算节点，作为数据服务运行。它们等待隐私计算请求，并在隐私计算中提供数据服务。


### 隐私计算客户端实现

隐私计算客户端基于客户端开发包(`mpc-proxy-sdk`)，实现了隐私计算发起，计算结果查询等，开发包详情参考[隐私客户端开发包](#privacy-contract-develop-manual#隐私客户端开发包)。

#### 创建maven工程

- 使用`IntelliJ`创建 JAVA 普通 Maven 工程，修改`pom.xml`，添加maven仓库地址

```xml
<repositories>
   <repository>
        <id>juzixmaven</id>
        <name>juzix maven</name>
        <url>https://sdk.platon.network/nexus/content/groups/public/</url>
    </repository>
</repositories>
```

- 引入`mpc-proxy-sdk`依赖。

```xml
<dependency>
    <groupId>net.platon.mpc</groupId>
    <artifactId>mpc-proxy-sdk</artifactId>
    <version>1.0</version>
</dependency>
```

- 添加隐私合约代理类

新建`platon.mpc.proxy` 包，将`plang`编译生成的`ProxyYaoMillionairesProblem.java`放入包中，该文件提供了隐私计算发起、计算结果查询等接口封装。

####　源码实现

基于`ProxyYaoMillionairesProblem`类创建程序入口函数，并完成隐私计算发起、计算结果查询。

>  **注意：算发起方者的钱包路径、钱包地址、解锁密码需要填写正确，并且钱包地址账户要有充足的energon**

main函数实现如下：

```java
public static void main(String[] args) {
    /*
    * 0. 需要提供以下参数
    */
    //计算发起方钱包文件，这里由 Alice 方发起请求
    String walletPath = "config/9a568e649c3a9d43b7f565ff2c835a24934ba447";
    //钱包密码
    String walletPass = "11111111";
    //任意节点的RPC端口地址
    String url = "http://127.0.0.1:6789";
    //发布的对应WASM合约地址
    String contractAddress = "0x43355c787c50b647c425f594b441d4bd75198888";

    /*
    * 1. 新建一个客户端
    */
    ProxyYaoMillionairesProblem client = new ProxyYaoMillionairesProblem(url, walletPath, walletPass);
    client.setContractAddress(contractAddress);

    /*
    * 2. 发起计算，并获取计算交易hash,如果失败会重试3次
    */
    String transactionHash = client.startCalc(ProxyYaoMillionairesProblem.Method.boolean_YaoMillionairesProblem_int_int, 3);
    
    /*
   * 3. 查询计算结果
   */
   String cipher = client.getResultByTransactionHash(transactionHash, 1000);
        if (cipher != null) {
            boolean b = client.getBool(cipher);
            logger.info("Client1 result : {} is richer", b ? "alice" : "bob");
        } else {
            logger.info("try later!");
        }
   }
}
```

`IntelliJ`直接运行main函数发起计算，控制台会输出发起计算的交易hash:

```
transaction hash: 0xa7423e579c6a6bbbc57d6201c6bef3f09944bad78c7036f0108fa27daef5ff6c
Client1 result : alice is richer
```

整个隐私计算完成，结果为：**Alice数据节点的输入值比Bob数据节点的输入值大**。

计算过程中数据节点之间不会泄密输入数据。

>  **注意：只有计算发起方能解密获取明文，其他人只能获取密文，且`getBool(cipher)` 方法转换会失败。**

更多细节与描述参考完整工程[mpc-proxy-sdk](https://github.com/PlatONnetwork/privacy-contract-sdk/blob/master/README.md)。

## 深入理解隐私合约编程

了解更多隐私合约开发细节，请参考：[深入理解隐私合约编程](zh-cn/development/[Chinese-Simplified]-%e6%b7%b1%e5%85%a5%e7%90%86%e8%a7%a3%e9%9a%90%e7%a7%81%e5%90%88%e7%ba%a6)。