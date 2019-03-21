# Privacy Contract Development Guide

## Introduction

The `PlatON` platform is the next-generation `Trustless` private data computing architecture based on secure multi-party computing for data privacy preserving features, using privacy contracts to implement privacy computing business logic, providing an infrastructure for privacy computing.


## Architecture

<div align=left>
<img src="zh-cn/development/privacy-contract/images/mpc_structure.png" width = "650" height="523"/>  
</div>
Architecture diagram description:

- PlatON Network

   A network of `PlatON` blockchain nodes, including compute nodes and other non-computing nodes.

- Privacy Contract

   Privacy contracts, deployed on blockchains, implement privacy computing business logic.

- Computation Node

   The compute node, as a `Platon` network node, contains a privacy algorithm to execute the virtual machine, providing a privacy computing runtime service.

- Data Node

   The data node, as a private data provider, assists the computing node in completing the privacy calculation service.

- Privacy Computation Client

   The privacy calculation client, as the privacy calculation request initiator, pays the calculation fee and is able to decrypt the privacy calculation result.

## Quick start

The privacy contract development is now presented in the classic case of MPC [Yao's Millionaire Problem](https://en.wikipedia.org/wiki/Yao%27s_Millionaires%27_Problem).

**Case statement**:

Two millionaires `Alice` and `Bob` want to know who two of them are richer, but they don't want to let each other know anything about their wealth.

**Solution**:

Using the `MPC` calculation, the two rich people can compare with the other party without knowing the specific property of the other party, and calculate who is the richest.

### Setting up the environment

- Setting up an environment reference: [Building a privacy development environment](en-us/development/privacy-contract-EN/_setting-up-privacy-computation-environment).

- Configure working directory:

   Create a working directory `YaoProblem`, install the privacy development environment compilation tool to this directory, and the workspace directory tree looks like this:

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

### Create a wallet

  For the privacy calculation participants, the privacy calculation requester, respectively create 3 wallet files and corresponding accounts:

**Windows command line**

```bash
> mkdir data
> platon.exe --datadir .\data account new
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat passphrase:
Address: {0x9a568e649c3a9d43b7f565ff2c835a24934ba447}Copy to clipboardErrorCopied
```

**Linux command line**

```bash
$ mkdir -p data
$ ./platon --datadir ./data account new
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat passphrase:
Address: {0x9a568e649c3a9d43b7f565ff2c835a24934ba447}Copy to clipboardErrorCopied
```

The above user work platform (Linux or Windows) performs 3 wallet creation operations, such as this sample output:

> ` 0x9a568e649c3a9d43b7f565ff2c835a24934ba447`

> `0xce3a4aa58432065c4c5fae85106aee4aef77a115`

> `0x9a568e649c3a9d43b7f565ff2c835a24934ba447`

Since the subsequent calculations need to consume `energon`, it is necessary to recharge the three wallet addresses separately.

- Testnet environment
  Apply for the `testnet` `energon` on [PlatON official website](https://developer.platon.network/#/energon?lang=zh).
- Private network environment
  Obtain `energon` from the private network consensus node `coinbase account`.

### Writing a privacy contract

Write a privacy contract `YaoMillionairesProblem.cpp` that solves `Yao’s millionaire problem`. code show as below:

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

### Compiling the privacy contract

#### Update configuration file

`config.json` is used to configure participant information, settlement rules, interface pricing, etc. in the calculation. Users need to update the configuration according to their own calculation requirements. This sample configuration is as follows:

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

Configuration file description:

-  `invitor` The inviting party's wallet address in the invitor data node.

- `parties` A list of wallet addresses corresponding to the two data nodes.
- `method-price` MPC algorithm function pricing (or charging), in the example the function `YaoMillionairesProblem` is priced as **200000**.
- `profit-rules` Specify the charge allocation rules after the calculation is completed, and allocate them according to the ratio. In the example, the allocation ratio is 1:2.
- `urls` Specify the participants' MPC service address information of the calculation. The key of each element is the wallet address of data provider, and the value is the MPC service address information (ICE specification address, where `DirectNodeServer` is fixed, `-h` and `-p` are node address and MPC service port respectively).

#### Compiling

The compile command is as follows,

**Linux platform**:

```bash
$ ./bin/plang ./YaoMillionairesProblem.cpp -config ./config.json -I ./include -mpcc YaoMillionairesContract.cpp
```

**Windows platform**：

```bash
> .\bin\plang YaoMillionairesProblem.cpp -config config.json -I .\include -mpcc YaoMillionairesContract.cpp
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

After compiling, it will generate a WASM contract, Java privacy contract proxy class, and other readme files, the directory tree are as follows:

```
├── code
│ └── java
│ ├── MPCYaoMillionairesProblem.java # For data providers
│ ├── MPCYaoMillionairesProblem-README.TXT
│ ├── ProxyYaoMillionairesProblem.java # For the calculation initiator
│ └── ProxyYaoMillionairesProblem-README.TXT
└── YaoMillionairesContract.cpp # Generate WASM contract, final contract file
```

For more details on the `plang` compiler, refer to [Usage](https://github.com/PlatONnetwork/privacy-contract-compiler#Usage).

**Note: Any updates to the configuration file or privacy contract code will need to be recompiled to take effect. **

### Privacy contract release

The following demonstrates the release of a Wasm contract on Ubuntu to complete the privacy contract release process. For more information on Wasm contract development, please refer to [Wasm Contract Development Guide](en-us/development/[English]-Wasm-Contract-Development-Guide).

1. Download the Ubuntu version [Wasm Contract Development Kit](https://download.platon.network/0.4/pwasm-linux-amd64-0.4.0.tar.gz) and extract it to ${pWasm}. If you have already downloaded, skip to the next step.
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
  "url": "http://127.0.0.1:6789",
  "gas": "0x99999788888",
  "gasPrice": "0x333330",
  "from":"0x9a568e649c3a9d43b7f565ff2c835a24934ba447"
}
```

> **Note: The release contract can be initiated by any account. **

5. Connect the `http://127.0.0.1:6789` node to unlock the `0x9a568e649c3a9d43b7f565ff2c835a24934ba447` account.

```bash
$ platon
> personal.unlockAccount("your-account")
Unlock account 0x9a568e649c3a9d43b7f565ff2c835a24934ba447
Passphrase:
True
```

6. Publish the Wasm contract.

```bash
$ chmod u+x ctool
$ ./ctool deploy --abi ./YaoProblem.cpp.abi.json --code ./YaoProblem.wasm --config ./config.json
```

Assume that the contract address after the contract is issued is:

> 0x43355c787c50b647c425f594b441d4bd75198888

### Data Node Service Implementation

Developers need to use the Privacy Data Service Development Kit (`mpc-data-sdk`) to build private data services. For more details, refer to [Privacy Data Services Development Kit](en-us/development/[English]-Deep-Understanding-Privacy-Contract-Dev).

#### Project construction

> `mpc-data-sdk` only provides the `JAVA` version, so the reader needs to have a certain JAVA programming foundation.

Create two Maven projects using `Intellij`, one is Alice data node service program, and the other is Bob data node service program. Do the following:

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

- Introducing `mpc-data-sdk` dependency

```xml
<dependency>
    <groupId>net.platon.mpc</groupId>
    <artifactId>mpc-data-sdk</artifactId>
    <version>1.0</version>
</dependency>
```

- Code implementation

Create a new `net.platon.vm.mpc` package in the project, add the `MPCYaoMillionairesProblem.java` to the `net.platon.vm.mpc` package (***Note***: **Do not modify the package name!**), then implement the `inputImpl` method for the Alice data node service program and the Bob data node service program respectively.

#### Interface Implementation

**Alice Data Node Service Program: **

Implement the `inputImpl` method in the `MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_01` class. The code is implemented as follows:

```java
/**
 * YaoMillionairesProblem(int,int)
 */
Final class MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_01 extends mpc_f_0588f14217b11e0f77e50d03a88ba866_01 {
    Public byte[] inputImpl(final InputRequestPara para) {
        Int ret_value = 0;
        // TODO: assemble data

        Ret_value = 1000000; // For Example Alice Input 1 million

        Return Data.Int32(ret_value);
    }
}
```

**Bob Data Node Service Program: **

Implement the methods in the `MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_02` class, the code is implemented as follows:

```java
/**
 * YaoMillionairesProblem(int,int)
 */
Final class MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_02 extends mpc_f_0588f14217b11e0f77e50d03a88ba866_02 {
    Public byte[] inputImpl(final InputRequestPara para) {
        Int ret_value = 0;
        // TODO: assemble data

        Ret_value = 2000000; // For Example Bob Input 2 million

        Return Data.Int32(ret_value);
    }
}
```

Class name format description:

`MPCYaoMillionairesProblem_YaoMillionairesProblem_int_int_01`

```
- MPCYaoMillionairesProblem MPC prefix + algorithm function name in privacy contract
- YaoMillionairesProblem method name
- int_int type of two input parameters
- 01 Data provider number, Alice is 01, Bob is 02
```

`mpc_f_0588f14217b11e0f77e50d03a88ba866_01`

```
- mpc_f specific prefix
- 0588f14217b11e0f77e50d03a88ba866 Method The MD5 value of the YaoMillionairesProblem function signature
- 01 Data provider number, Alice is 01, Bob is 02
```

> **Note: The data node program MUST implement the `InputImpl` method of the proxy class**

#### Implementation program entry main

Both projects add data node service program entry classes as follows:

```java
Public class Client {
    Public static void main(String[] args) {
        ConfigInfo cfgInfo = new ConfigInfo(args);
        // Here you can hardcode the parameters in cfgInfo
        // cfgInfo.walletPath=""
        // cfgInfo.walletPass=""
        // cfgInfo.iceCfgFile=""
        AppClient app = new AppClient(cfgInfo);
        App.start(args);
    }
}
```

The program needs to pass in 3 parameters, the parameter description:

| Parameter  | Description                            | Available |
| :--------- | :------------------------------------- | :-------- |
| walletPath | Privacy Data Provider Wallet File Path | No        |
| walletPass | Account Password                       | No        |
| iceCfgFile | ICE Profile                            | No        |

#### Packaged Data Node Program

Package the two engineering projects separately, and execute the path where the project `pom.xml` is located:

```bash
$ mvn build
```

The two data node service programs output: `mpc-data-sdk-client1-1.0-SNAPSHOT.jar`, `mpc-data-sdk-client2-1.0-SNAPSHOT.jar`.

#### Configuring the data node program

This example assumes that it is built as a private network and implemented on the same machine.

The `Alice` data node creates the configuration file `cfg.client1.config` with the following contents:

```config
TaskCallback.Proxy=tasksession:default -h 127.0.0.1 -p 10001
Callback.Client.Endpoints=default -h 127.0.0.1
```

The `Bob` data node creates the configuration file `cfg.client2.config` with the following contents:

```config
TaskCallback.Proxy=tasksession:default -h 127.0.0.1 -p 10002
Callback.Client.Endpoints=default -h 127.0.0.1
```

Configuration file description:

- The address and port of `TaskCallback.Proxy` and [Startup with MPC calculation function](en-us/basics/[English]-Private-Networks#Enabling-MPC-for-a-node) In the `PlatON` node, the address and port configured in the ICE configuration file specified by the parameter `--mpc.ice` are the same (the default port is 8201).
- `Callback.Client.Endpoints` is the server address where the data privacy service is started.

#### The program runs

Run the Alice data node:

```bash
$ java -jar mpc-data-sdk-client1-1.0-SNAPSHOT.jar --walletPath=./config/9a568e649c3a9d43b7f565ff2c835a24934ba447 --walletPass=11111111 --iceCfgFile=./config/cfg.client1.config
```

Run the Bob data node:

```bash
$ java -jar mpc-data-sdk-client2-1.0-SNAPSHOT.jar --walletPath=./config/ce3a4aa58432065c4c5fae85106aee4aef77a115 --walletPass=11111111 --iceCfgFile=./config/cfg.client2.config
```

After the data node program is started, it will be registered to the computation node and run as a data service. They wait for privacy calculation requests and provide data services in privacy calculations.

### Privacy Computing Client Implementation

The privacy computing client is based on the client development kit (`mpc-proxy-sdk`), which implements privacy calculation initiation, calculation result query, etc., and the development package details refer to [Privacy Client Development Kit] (#privacy-contract-develop-manual #私客户端开发包).

#### Creating a maven project

- Create a JAVA normal Maven project with `IntelliJ`, modify `pom.xml`, add the maven repository address

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

- Add privacy contract proxy class

Create a new `platon.mpc.proxy` package and put `ProxyYaoMillionairesProblem.java` generated by `plang` into the package. This file provides interface encapsulation such as privacy calculation initiation and calculation result query.

#### Source implementation

Create a program entry function based on the `ProxyYaoMillionairesProblem` class, and complete the privacy calculation initiation and calculation result query.

> **Note: The originator's wallet path, wallet address, and unlock password need to be filled in correctly, and the wallet address account should have sufficient energon**

The main function is implemented as follows:

```java
Public static void main(String[] args) {
    /*
    * 0. The following parameters are required
    */
    / / Calculate the originator wallet file, where the request is initiated by the Alice side
    String walletPath = "config/9a568e649c3a9d43b7f565ff2c835a24934ba447";
    //wallet password
    String walletPass = "11111111";
    / / RPC port address of any node
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
    
    /*
   * 3. Query calculation results
   */
   String cipher = client.getResultByTransactionHash(transactionHash, 1000);
        If (cipher != null) {
            Boolean b = client.getBool(cipher);
            Logger.info("Client1 result : {} is richer", b ? "alice" : "bob");
        } else {
            Logger.info("try later!");
        }
   }
}
```

`IntelliJ` directly runs the main function to initiate the calculation, and the console will output the transaction hash that initiated the calculation:

```bash
Transaction hash: 0xa7423e579c6a6bbbc57d6201c6bef3f09944bad78c7036f0108fa27daef5ff6c
Client1 result : alice is richer
```

The entire privacy calculation is completed, and the result is: **Alice data node input value is larger than the Bob data node input value**. Meanwhile the input data is not leaked between data nodes during the calculation.

> **Note: Only the calculation initiator can decrypt the plaintext, others can only get the ciphertext, and the `getBool(cipher)` method conversion will fail. **

For more details and descriptions please refer to the full project [mpc-proxy-sdk](https://github.com/PlatONnetwork/privacy-contract-sdk/blob/master/README.md).

## Deep understanding of privacy contract programming

For more details on privacy contract development, please refer to: [In-depth understanding of privacy contract programming](en-us/development/[English]-Deep-Understanding-Privacy-Contract-Dev).

