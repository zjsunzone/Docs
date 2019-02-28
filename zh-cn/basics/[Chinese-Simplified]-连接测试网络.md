
> 本文档将说明怎么将本地`PlatON`节点和测试网络链接。

贝莱世界测试网络现已开放，按照以下步骤可将本地节点连接到测试网络。

设置前确保本地已经按照[PlatON安装指南](zh-cn/basics/[Chinese-Simplified]-安装指南)安装好PlatON节点。

本文假设Ubuntu环境下工作目录为 `~/platon-node` ，Windows环境下工作目录为 `D:\platon-node`。注意后续均在工作目录下进行。

## 创建账号

通过`Platon`命令，可创建账户:

- Windows命令行

```
D:\platon-node> mkdir data
D:\platon-node> platon.exe --datadir .\data account new
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat passphrase:
Address: {550ae58b051a8e942f858ef22019c1c622292f7e}
```

- Linux命令行：

```
$ mkdir -p data
$ ./platon --datadir ./data account new
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase:
Repeat passphrase:
Address: {550ae58b051a8e942f858ef22019c1c622292f7e}
```

输出结果为生成的账户地址。

## 申请测试Energon

使用第一步生成的账户地址在[PlatON官网](https://developer.platon.network/#/energon?lang=zh)申请测试Energon。

***注意：测试Energon没有任何价值，仅限于体验测试网络功能。如仅仅只是连接测试网络，无需申请！***


## 启动本地节点并连接测试网络
 
- Windows命令行

```
D:\platon-node> platon.exe --identity platon --datadir .\data --port 16789 --rpcport 6789 --rpcapi "db,eth,net,web3,admin,personal" --rpc --debug --verbosity 3 --rpcaddr 0.0.0.0  --syncmode "full" --gcmode "archive" --testnet
```

- Linux命令行

```
$ ./platon --identity platon --datadir ./data --port 16789 --rpcport 6789 --rpcapi "db,eth,net,web3,admin,personal" --rpc --debug --verbosity 3 --rpcaddr 0.0.0.0  --syncmode "full" --gcmode "archive" --testnet
```

***提示：***

| 选项         | 描述                     |
|:------------ |:------------------------ |
| --identity   | 指定网络名称             |
| --datadir    | 指定data目录路径         |
| --rpcaddr    | 指定rpc服务器地址        |
| --rpcport    | 指定rpc协议通信端口      |
| --rpcapi     | 指定节点开放的rpcapi名称 |
| --rpc        | 指定http-rpc通讯方式     |
| --testnet    | 连接到测试网络          |

更多参数意义通过`platon --help`命令查看。

## 查看是否添加成功

1.通过`http`方式进入`platon`控制台

- Windows命令行

```
D:\platon-node> platon.exe attach http://localhost:6789
```

- Linux命令行：

```
$ ./platon attach http://localhost:6789
```


2.查看节点列表是否添加测试网络

```
> admin.peers
[{
    caps: ["eth/62", "eth/63"],
    id: "23aa343260d06e04107d1cd9a7d12c54cc238719a1523ffe42640210c913218b5940d41511c5adb716da38844a85cdab8b7db0600d242e24168d7df10aebd324",
    name: "PlatONnetwork/V0.5_testsn/v0.4.0-stable-0f651de0/linux-amd64/go1.11",
    network: {
      consensus: false,
      inbound: false,
      localAddress: "192.168.18.181:51828",
      remoteAddress: "54.252.202.130:16789",
      static: false,
      trusted: false
    },
    protocols: {
      eth: {
        head: "0x104fe03d2b2f0b783e808ea7fcd52566d7cde9f36c4a06e950795e0459db5551",
        number: 74822,
        version: 63
      }
    }
},
...
]
```

节点列表中出现一系列测试网络节点，则表示连接成功！
