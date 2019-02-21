
## Introduction

As the next-generation Trustless secure data computing architecture, the [PlatON](#) platform uses Secure Multi-Party Computing (`MPC`) as a protocol for privacy computing, providing the infrastructure for privacy computing. The PlatON public-chain network and the external computing network are combined into a data-centric composite network to provide users with secure computing services to realize the flow of data values. On the `PlatON` platform, the **MPC Compute Virtual Machine**(`MPC VM`) is a key component of the PlatON computing architecture, which uses **privacy contracts** for `MPC` calculations and provides stable runtime services.


## Architecture

<div align=left>
<img src="zh-cn/development/privacy-contract/images/mpc_structure.png" width = "650" height="523"/>  
</div>

Architecture diagram description:

- PlatON

    Here is the `PlatON` blockchain node with `MPC` functionality.

- Privacy Contract

    Privacy contract, a special secure multi-party computing (`MPC`) contract developed in `C++` for operation in the MPC Execution Virtual Machine (`MPC VM`) at the PlatON node. The privacy contract uses the MPC capabilities provided by the PlatON node to protect the privacy of the computing data.

- MPC VM

    The `MPC` execution virtual machine, provides a privacy contract runtime environment, and executes MPC computing objects at the bottom to implement the MPC algorithm, which relies on the computing services provided by the PlatON node.

- Privacy App

    The privacy application client, as the privacy calculation initiator, initiates a calculation request to the node, queries the calculation result, and the like.

- Privacy Data Service

    The privacy data service, as a provider of privacy calculation data, provides data to the compute nodes. This service is associated with the MPC VM and participates in MPC computing.

- Data Source

    A data source that provides raw data streams (such as databases, files, network streams, etc.) for private data services and is the source of private data.

## Quick start

Now using the classic case of MPC, [Yao's millionaire problem](https://en.wikipedia.org/wiki/Yao%27s_Millionaires%27_Problem), to introduce privacy contract development.

**Case statement:**

Two millionaires, Alice and Bob, want to know who are richer, but they don't want to let each other know anything about their wealth.

**Solution:**

Using MPC calculations, two rich people can compare with each other without knowing the specific property of the other party, and calculate who is the richest.

### Build a development environment

Let's take the `Ubuntu16.04` operating system as an example to build a build environment. Let's assume that the `${privacy-contract-compiler}` directory is the Ubuntu directory `/home/platon/privacy-contract-compiler`, and `${pWasm}` is the Ubuntu directory `/home/platon/pWasm` , where `platon` is the currently logged user. The user can correspond to the user who logged in.

**Dependent components:**

- **PlatON cluster environment**

    The privacy contract operation relies on the Platon network cluster environment to provide the core functionality of the blockchain. To enable the node to provide MPC computing power, MPC computing must be enabled.

    1. Install the platon executable with MPC functionality, view [the installation guide](#).
    2. Set up the cluster environment, view [the PlatON cluster environment](#).
    3. Enable MPC calculations on the node, view [enable MPC calculations](#).

- **Protobuf 3.5.2**

    `Protobuf` is used in the privacy contract to implement custom structs and to provide pb message packaging and unpacking capabilities. The current protobuf needs to be a customized version of `3.5.2` and can be installed from [source](https://github.com/PlatONnetwork/privacy-contract-vm#building--installing).

- **Plang compiler**

    `Plang` is a custom compiler for `PlatON` that compiles privacy contracts to generate `Wasm smart contract` source code files and JAVA agents. The [privacy-contract-compiler](https://github.com/PlatONnetwork/privacy-contract-compiler#Build)  open source toolkit is available on the official website. After downloading, extract it to the `${privacy-contract-compiler} `directory and install it according to the `README`.

- **emp-tool library**

    Use [emp-tool](https://github.com/PlatONnetwork/emp-tool)  to provide the core MPC calculation object. Download the library using the following command.

    ```bash
    $ git clone https://github.com/PlatONnetwork/emp-tool.git
    ```

    The emp-tool/include directory contains the header files needed for contract development:

    ```bash
    include/
    ├── batcher.h
    ├── bit.h
    ├── block.h
    ├── comparable.h
    ├── constants.h
    ├── emp-tool.def
    ├── float_circuit.h
    ├── integer.h
    ├── number.h
    └── swappable.h
    ```

### Introduction to Privacy Service Development

The development of privacy services based on privacy contracts, the implementation mainly includes:

- [Privacy Contract Implementation](#privacy-contract-implementation)

    The main implementation is based on the `MP-C` algorithm written by `emp-toolkit`, which is then compiled and deployed to the `PlatON` platform.

- [Privacy Data Service Implementation](#private-data-service-implementation)

    It mainly implements the docking of `MPC VM`, provides private data for privacy calculation, and is developed based on the `Private Data Service Development Kit'.

- [Private Client Implementation](#private-client-implementation)

    It mainly implements the initiation of the privacy calculation request and the acquisition of the calculation result, and is developed based on the "private client development package".

### Privacy contract implementation

#### Privacy Contract Coding

Write a privacy contract `YaoMillionairesProblem.cpp` to solve Yao's millionaire problem. The code is implemented as follows:

```c++
#include <iostream>
#include "integer.h" // Use the emp-tool library header file

/*
  * Yao's millionaires' problem, show who is richer.
  * @param money1, Alice participant
  * @param money2, Bob participant
  * @return true if Alice >= Bob else false.
  */
Bool YaoMillionairesProblem(int money1, int money2) {
     Std::cout << __FUNCTION__
         << " Alice: " << money1
         << " Bob: " << money2
         << std::endl;

     Emp::Integer alice_money(money1, emp::ALICE); // Set Alice amount
     Emp::Integer bob_money(money2, emp::BOB); // Set the Bob amount

     Int ret = (alice_money - bob_money).reveal(); // Execute the comparison size and get the comparison result
     Std::cout << __FUNCTION__ << " result(=Alice-Bob): " << ret << std::endl;

     Return ret >= 0;
}
```

Currently, the privacy contract project needs to create a project directory under the `plang` project after `make`, and then into the `{privacy-contract-compiler}/build/bin` directory, here is `YaoProblem`.

```bash
$ pwd
/home/platon/privacy-contract/privacy-contract-compiler/build/bin
$ mkdir YaoProblem && cd YaoProblem
```

Save the above code as the file `YaoMillionairesProblem.cpp` and save the code file to the `YaoProblem` directory.

#### Privacy Contract Compilation

Before compiling the privacy contract, copy `emp-tool/include` to the project directory `YaoProblem` and look at the workspace directory tree as follows:

```bash
.
├── clang                           # clang compiler
├── plang                           # plang compiler
├── YaoProblem                      # Contract Development Directory
|   ├── config.json                 # configuration file
|   ├── YaoMillionairesProblem.cpp  # Privacy Agreement
|   └── include                     # emp-tool library header file
|       ├── integer.h               # privacy calculation object header file
|       └── ...
└── ...
```

`config.json` is a configuration file that configures the basic information of two data providers. The user needs to change the account address of the data service. The configuration is as follows:

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
        "0x60ceca9c1290ee56b98d4e160ef0453f7c40d219": 1,
        "0x3771c08952f96e70af27324de11bb380ec388ec3": 2
    },
    "urls": {
        "0x60ceca9c1290ee56b98d4e160ef0453f7c40d219": "DirectNodeServer:default -h 127.0.0.1 -p 10001",
        "0x3771c08952f96e70af27324de11bb380ec388ec3": "DirectNodeServer:default -h 127.0.0.1 -p 10002"
    }
}
```

Config file field description:

- `invitor` : Address of computation originator; is one of `parties`.
- `parties`: List of computation participator addresses; currently only supports two parties.
- `method-price`: Prices for contract methods, in the example, the function `YaoMillionairesProblem` is priced as **200000**.
- `profit-rules`: Profit distribution ratio.
- `urls`: Addresses of MPC servers participating in the computation. The contents is a JSON object where the keys are the addresses of the participants and the values are strings used to set up the MPC server addresses. The latter consist of the fixed string `DirectNodeServer` and the parameters `-h` and `-p` used to set the node's address and MPC server port. The MPC port is set [here](%English%5d-%e7%a7%81%e6%9c%89%e7%bd%91%e7%bb%9c#Enable MPC ability in a node).

Compile command:

```shell
$ ../plang ./YaoMillionairesProblem.cpp -config ./config.json -I ./include -mpcc YaoMillionairesContract.cpp
```

Terminal output:

```
digest:
 * IR NAME: MPCYaoMillionairesProblem
 * IR HASH: be4bfbbc708c5622e0ff6f274020d654
 * <p>
 * IR FUNC HASH(MD5)                 IR FUNC NAME            IR FUNC PROT
 * 0588f14217b11e0f77e50d03a88ba866  YaoMillionairesProblem  YaoMillionairesProblem(int,int)
```

The output file is as follows:

```
├── code
│   └── java
│       ├── MPCYaoMillionairesProblem.java              # For data providers
│       ├── MPCYaoMillionairesProblem-README.TXT
│       ├── ProxyYaoMillionairesProblem.java            # For the calculation initiator
│       └── ProxyYaoMillionairesProblem-README.TXT
└── YaoMillionairesContract.cpp                         # Generate WASM contract, final contract file
```

For more details on the `plang` compiler, refer to [Usage](https://github.com/PlatONnetwork/privacy-contract-compiler#Usage).

**Note: Any updates to the configuration file or privacy contract code will need to be recompiled to take effect.**

#### Privacy contract release

As a chain service that provides privacy calculations, the privacy contract will eventually be compiled into the `PlatON` network, which must be compiled into [Wasm contract](#) file, and then compile and release the Wasm contract to finally achieve the privacy contract release.

The following is a brief description of the release of the Wasm contract `YaoMillionairesContract` generated on Ubuntu to complete the privacy contract release process. For more information on Wasm contract development under `Ubuntu`, please refer to [Wasm Contract Development Guide](#).


1. Download the Ubuntu version [Wasm Contract Development Kit 0.4](https://download.platon.network/0.4/pwasm-linux-amd64-0.4.0.tar.gz) and extract it to ${pWasm}. If you have already downloaded, skip to the next step.

2. Go to the ${pWasm} directory and execute the following command to generate the Wasm contract project `YaoProblem`.

```bash
$ cd {pWasm}
$ ./script/autoproject.sh . YaoProblem
```

3. Copy the compiled file `YaoMillionairesContract.cpp` from the privacy contract to `{pWasm}/user/YaoProblem/YaoProblem.cpp` and go to `{pWasm}/build` to compile.

```bash
$ cp YaoMillionairesContract.cpp ${pWasm}/user/YaoProblem/YaoProblem.cpp
$ cd {pWasm}/build
$ make
```

4. Create a `config.json` configuration file to configure the information published by the current contract:

```bash
{
  "url":"http://127.0.0.1:6789",
  "gas":"0x99999788888",
  "gasPrice":"0x333330",
  "from":"0x9a568e649c3a9d43b7f565ff2c835a24934ba447"
}
```

**Note: The release contract can be initiated by any account.**

5. Connect the `http://127.0.0.1:6789` node to unlock the `0x9a568e649c3a9d43b7f565ff2c835a24934ba447` account.

```bash
$ platon 
> personal.unlockAccount("your-account")
Unlock account 0x9a568e649c3a9d43b7f565ff2c835a24934ba447
Passphrase:
true
```

6. Publish the Wasm contract.

```bash
$ chmod u+x ctool
$ ./ctool deploy --abi ./YaoProblem.cpp.abi.json --code ./YaoProblem.wasm --config ./config.json
```

Assume that the contract address after the contract is issued is:

> 0x43355c787c50b647c425f594b441d4bd75198888


### Privacy Data Service Implementation

By introducing the Privacy Data Service Development Kit (`mpc-data-sdk`), the data provider can register its own private data service to the `MPC VM` and provide calculation data. For more details, refer to the [Private Data Service Development Kit](#).

**Private Data Provider & MPC Compute Node:**

The privacy data provider's wallet file is saved on the MPC compute node, and the data provider's private data service must be registered to the node's `MPC VM`. Because privacy calculations are multi-party calculations, there are currently only two-party calculations, distinguished by `Alice` and `Bob`. When compiling the privacy contract, the `Alice` party address is configured as `0x9a568e649c3a9d43b7f565ff2c835a24934ba447`, and the `Bob` party address is `0xce3a4aa58432065c4c5fae85106aee4aef77a115`.

**Step 1: Write a private data service code**

Currently `mpc-data-sdk` only provides the `JAVA` version, so the reader needs to have a certain JAVA programming foundation. The following steps will demonstrate how to access `mpc-data-sdk` in the simplest way, some of which need to be done manually.

* Create a JAVA normal Maven project, modify `pom.xml`, add the maven repository address

```xml
<repositories>
   <repository>
        <id>juzixmaven</id>
        <name>juzix maven</name>
        <url>https://sdk.platon.network/nexus/content/groups/public/</url>
    </repository>
</repositories>
```

* Introducing `mpc-data-sdk` dependency

```xml
<dependency>
    <groupId>net.platon.mpc</groupId>
    <artifactId>mpc-data-sdk</artifactId>
    <version>1.0</version>
</dependency>
```

* Code implement

Create a new `net.platon.vm.mpc` package in the project and put the JAVA code `MPCYaoMillionairesProblem.java` generated by `plang` to compile the privacy contract into the package. **Do not modify the package name!!** Next implement the `inputImpl` method.

The Alice side only needs to implement the methods in the `MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_01` class. The code is implemented as follows:

```java
/**
 * YaoMillionairesProblem(int,int)
 */
final class MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_01 extends mpc_f_0588f14217b11e0f77e50d03a88ba866_01 {
    public byte[] inputImpl(final InputRequestPara para) {
        int ret_value = 0;
        // TODO: assemble data

        ret_value = 1000000; // Alice 1 million

        return Data.Int32(ret_value);
    }
}
```

The Bob side only needs to implement the methods in the `MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_02` class. The code is implemented as follows:

```java
/**
 * YaoMillionairesProblem(int,int)
 */
final class MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_02 extends mpc_f_0588f14217b11e0f77e50d03a88ba866_02 {
    public byte[] inputImpl(final InputRequestPara para) {
        int ret_value = 0;
        // TODO: assemble data

        ret_value = 2000000; // Bob 2 million

        return Data.Int32(ret_value);
    }
}
```

The file name and function name appear in the file are exactly the same and are not short, but this is in line with the naming convention. So don't confuse the class name or the method name is too long, the class name format description:

`MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_01`

    MPCYaoMillionairesProblem           MPC prefix + algorithm function name in privacy contract
    YaoMillionairesProblem              method name
    int_int                             type of two input parameters
    01                                  data provider number, Alice is 01, Bob is 02

`mpc_f_0588f14217b11e0f77e50d03a88ba866_01`

    mpc_f                               specific prefix
    0588f14217b11e0f77e50d03a88ba866    method The MD5 value of YaoMillionairesProblem, MD5 is used here for future function overloading.
    01                                  data provider number, Alice is 01, Bob is 02

**Note: The data provider MUST implement the `InputImpl` method in the proxy class, otherwise the data obtained during the calculation is the default value of the associated type.**

* Define program entry

Added application entry class, which is used to accept parameters and start the service when launching the application:

```java
public class Client {
    public static void main(String[] args) {
        ConfigInfo cfgInfo = new ConfigInfo(args);
        // Here you can hardcode the parameters in cfgInfo
        // cfgInfo.walletPath=""
        // cfgInfo.walletPass=""
        // cfgInfo.iceCfgFile=""
        AppClient app = new AppClient(cfgInfo);
        app.start(args);
    }
}
```

The program needs to pass in 3 parameters, which can be passed in via the `args` system variable or hardcoded in the `main` function.

`main` function into the parameter description:

| Parameter  | Description                            | Empty |
| :--------- | :------------------------------------- | :---- |
| walletPath | Privacy data provider wallet file path | No    |
| walletPass | Account password                       | No    |
| iceCfgFile | ICE profile                            | No    |

**Step 2: Configure related files**

The `cfg.client1.config` of the `Alice` side is configured as follows:


```
TaskCallback.Proxy=tasksession:default -h 127.0.0.1 -p 10001
Callback.Client.Endpoints=default -h 10.10.8.163
```

The `cfg.client2.config` of the `Bob` side is configured as follows:

```
TaskCallback.Proxy=tasksession:default -h 127.0.0.1 -p 10002
Callback.Client.Endpoints=default -h 10.10.8.163
```

Configuration file description:

- Two data providers (`Alice` & `Bob`) are required in the MPC calculation process to start the private data service at the same time, but the `Alice` and `Bob` sides are not started in the order.
- The address and port of `TaskCallback.Proxy` and [Startup with MPC calculation function](#) In the PlatON node, the address and port configured in the ICE configuration file specified by the parameter `--mpc.ice` are the same.
- If the configuration of `--mpc.ice` is not specified when starting the node, the default port is `8201`.
- `Callback.Client.Endpoints` is the server address where the data privacy service is started.


**Step 3: Packing and running**

You can run the maven project as an executable jar package, execute it in the path of the project `pom.xml`

```bash
$ mvn build
```

You can also execute the main function directly. We run as a jar package with the following commands:

Alice side:

```bash
$ java -jar mpc-data-sdk-client1-1.0-SNAPSHOT.jar --walletPath=./config/9a568e649c3a9d43b7f565ff2c835a24934ba447 --walletPass=11111111 --iceCfgFile=./config/cfg.client1.config
```

Bob side:

```bash
$ java -jar mpc-data-sdk-client2-1.0-SNAPSHOT.jar --walletPath=./config/ce3a4aa58432065c4c5fae85106aee4aef77a115 --walletPass=11111111 --iceCfgFile=./config/cfg.client2.config
```

The above command will cause the program to connect to the MPC compute node when it starts. It should be noted that only the data provider and the respective `MPC` compute nodes are successfully connected to proceed to the next step.


### Privacy Client Implementation

By introducing the privacy client development kit (`mpc-proxy-sdk`), users can build their own privacy client to initiate calculations, query calculation results, etc. Development package interface details refer to [Privacy Client Development Kit](#).

precondition:

1. Two MPC compute nodes have been running normally;
2. The privacy data service program has been successfully connected to each computing node;
3. Be prepared to calculate the originator's wallet and password;
4. The contract address of the `WASM` contract on the chain has been obtained;

After confirming that the above four conditions have been met, start the following process:

**Step 1: Privacy client implementation code**

- Create a JAVA normal Maven project, modify `pom.xml`, add the maven repository address

```xml
<repositories>
   <repository>
        <id>juzixmaven</id>
        <name>juzix maven</name>
        <url>https://sdk.platon.network/nexus/content/groups/public/</url>
    </repository>
</repositories>
```

- Introducing the `mpc-proxy-sdk` dependency.

```xml
<dependency>
    <groupId>net.platon.mpc</groupId>
    <artifactId>mpc-proxy-sdk</artifactId>
    <version>1.0</version>
</dependency>
```

- Code implementation

Create a new `platon.mpc.proxy` package in the project, compile `plang` and generate `ProxyYaoMillionairesProblem.java` in the JAVA code to create the program entry. The main function is implemented as follows:

```java
public static void main(String[] args) {
    /*
    * 0. The following parameters are required
    */
    //Calculate the originator wallet file, where the request is initiated by the Alice side
    String walletPath = "config/9a568e649c3a9d43b7f565ff2c835a24934ba447";
    //wallet password
    String walletPass = "11111111";
    //RPC port address of any node
    String url = "http://127.0.0.1:6789";
    //The corresponding WASM contract address issued
    String contractAddress = "0x43355c787c50b647c425f594b441d4bd75198888";

    /*
    * 1. Create a new client
    */
    ProxyYaoMillionairesProblem client = new ProxyYaoMillionairesProblem(url, walletPath, walletPass);
    client.setContractAddress(contractAddress);

    /*
    * 2. Initiate the calculation and get the calculated transaction hash, if it fails, try again 3 times
    */
    String transactionHash = client.startCalc(ProxyYaoMillionairesProblem.Method.boolean_YaoMillionairesProblem_int_int, 3);

}
```

**Step 2: Initiate a calculation request**

In this step, for debugging purposes, you can run the main function directly on the IDE to initiate the calculation. At this time, the transaction hash that initiates the calculation is output in the console:

```
Transaction hash: 0xa7423e579c6a6bbbc57d6201c6bef3f09944bad78c7036f0108fa27daef5ff6c
```

**Step 3: Query results**

Can be obtained by the method provided in sdk, the code is implemented as follows:

```java
public static void main(String[] args) {
    /*
    * 0. The following parameters are required
    */
    //Calculate the originator wallet file, where the request is initiated by the Alice side
    String walletPath = "config/9a568e649c3a9d43b7f565ff2c835a24934ba447";
    //wallet password
    String walletPass = "11111111";
    //RPC port address of any node
    String url = "http://127.0.0.1:6789";
    //The corresponding WASM contract address issued
    String contractAddress = "0x43355c787c50b647c425f594b441d4bd75198888";

    /*
    * Create a new client
    */
    ProxyYaoMillionairesProblem client = new ProxyYaoMillionairesProblem(url, walletPath, walletPass);
    client.setContractAddress(contractAddress);

   /*
    * By initiating a transaction hash, querying results, timeout 1s
    * transactionHash shows the transaction in the console for the second step.
   */
   String cipher = client.getResultByTransactionHash(transactionHash, 1000);
        if (cipher != null) {
            boolean b = client.getBool(cipher);
            logger.info("Client1 result : {} richer", b ? "alice" : "bob");
        } else {
            logger.info("try later!");
        }
   }
}
```

The result `cipher` returned by the transaction hash is the ciphertext, and the `getBool(cipher)` method decrypts and converts the ciphertext to the appropriate type.

**Note: The plaintext can only be decrypted by the calculation initiator in the previous step. Others can only get the ciphertext. The conversion with the `getBool(cipher)` method will fail.**

Query transaction receipt:

```bash
$ ./platon attach http://localhost:7789
> eth.getTransactionReceipt("0xa7423e579c6a6bbbc57d6201c6bef3f09944bad78c7036f0108fa27daef5ff6c")
```

The results are as follows:

```json
{
  blockHash: "0x7b59c274d6e4e67cbee9e92e754ada44e6ff88fe4de632680758a3d6eb9eecc0",
  blockNumber: 4977,
  contractAddress: null,
  cumulativeGasUsed: 112767,
  from: "0x60ceca9c1290ee56b98d4e160ef0453f7c40d219",
  gasUsed: 112767,
  logs: [{
      address: "0xb136a7190dd23c57c1d8f07011174af3bbe3cfe2",
      blockHash: "0x7b59c274d6e4e67cbee9e92e754ada44e6ff88fe4de632680758a3d6eb9eecc0",
      blockNumber: 4977,
      data: "0xf84301b84064396361306232363866376163336339323732313565666234636335653038393339336631376631663336363765656666313664396536653836313866353137",
      logIndex: 0,
      removed: false,
      topics: ["0x9938524d0b7638d8cb06ef86cc46ae3074007aea703ce237efe1470b6bdb5f31"],
      transactionHash: "0xa7423e579c6a6bbbc57d6201c6bef3f09944bad78c7036f0108fa27daef5ff6c",
      transactionIndex: 0
  }],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000020000000000000000000000010000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000100000000000000000000000004000000000000000000000000000000000000000000000000000000000000000000000800000000000000000000000000000000",
  status: "0x1",
  to: "0xb136a7190dd23c57c1d8f07011174af3bbe3cfe2",
  transactionHash: "0xa7423e579c6a6bbbc57d6201c6bef3f09944bad78c7036f0108fa27daef5ff6c",
  transactionIndex: 0
}
```

In the result receipt, if the log is empty, the task execution fails! Refer to the full project [mpc-proxy-sdk](https://github.com/PlatONnetwork/privacy-contract-sdk/blob/master/README.md) for more details and descriptions.

## Deep understanding of privacy contract programming

For more details on privacy contract development, please refer to: [In-depth understanding of privacy contract programming]([English]-Deep-Understanding-Privacy-Contract-Dev).

