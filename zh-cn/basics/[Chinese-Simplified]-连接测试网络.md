
> 本文档将说明怎么将本地`PlatON`节点和测试网络链接。

贝莱世界测试网络现已开放，按照以下步骤可将本地节点连接到测试网络。

设置前确保本地已经按照[PlatON安装指南](./[Chinese-Simplified]-安装指南)安装好PlatON节点。

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

***注意：测试Energon没有任何价值，仅限于体验测试网络功能***

## 创建创世区块配置文件

在工作目录下创建文件`platon.json`，文件内容如下：

```
{
	"config": {
		"chainId": 100,
		"homesteadBlock": 1,
		"eip150Block": 2,
		"eip150Hash": "0x0000000000000000000000000000000000000000000000000000000000000000",
		"eip155Block": 3,
		"eip158Block": 3,
		"byzantiumBlock": 4,
		"cbft": {
			"initialNodes":  [
			   "enode://37771debfe76b9335b4e6ae96d7a611cc82a57373c73f0381822467389ae2b521325b755aacc71a66f26821bb83e231e0a87ed0e92d3e1a97af5963eb87063bd@3.0.225.115:16790",
			   "enode://6ddd600a544ae4cfd5b4f2b0879afce79234f9755e70fb16b0385b9094a161e328a20074044dcc7b80ea50b5929ad38417c7e3e2d550686250ed40f4f98f3dee@3.0.225.115:16791",
			   "enode://db862efe5974899e138d79c446567f7687184c0d76a20a9c4f9ed4b68f62fa15ae6a1adfd70cd57b870ef969f4fe7ef4225ecf64de3732389523dee87ed177e2@35.156.19.166:16790",
			   "enode://84b1daf160bcdbdce5cac0779eacd5aae2ff94dbd01ae86dd21d8307238687a78a5bcf9e446779fe3bffd206e24f43da5905a5fa4b80786beb6bea44447a9755@35.156.19.166:16791",
               "enode://1d7f364569aafa1520ceb94181ff87d63c3e8354e192d17ac4535d25cd41d857fa3fc7f388648bf092cebf5839cf93ac97e2e04d886cd31b858bb274f44da506@54.250.159.27:16790",
			   "enode://ee16e3a652fff1092c935d6b399cabfa69f4bb16a7cd06fe2d7907c50b682efcd76635dcbe5c6281943657478e897a8ed03c5577d3f817c15c8eb9eeb50bd215@54.67.96.43:16790",
		       "enode://b70e884cfad9443898ba0e28a45c0e14332cf46dfbb8a9a07983b909d99cd0586af2cc5ff579b4352170e42320a254cb57b84eb97d7be86a1bd48a058683033b@54.67.96.43:16791"
        ]
		}
	},
	"nonce": "0x0",
	"timestamp": "0x5bc94a8a",
	"extraData": "0x00000000000000000000000000000000000000000000000000000000000000007a9ff113afc63a33d11de571a679f914983a085d1e08972dcb449a02319c1661b931b1962bce02dfc6583885512702952b57bba0e307d4ad66668c5fc48a45dfeed85a7e41f0bdee047063066eae02910000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
	"gasLimit": "0x99947b760",
	"difficulty": "0x1",
	"mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
	"coinbase": "0x60ceca9c1290ee56b98d4e160ef0453f7c40d219",
	"alloc": {
		"1fe1b73f7f592d6c054d62fad1cc55756c6949f9": {
			"balance": "0x295be96e640669720000000"
		 }
	},
	"number": "0x0",
	"gasUsed": "0x0",
	"parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```

## 初始化本地节点

- Windows命令行

```
D:\platon-node> platon.exe --datadir ./data init platon.json
```

- Linux命令行

```
$ ./platon --datadir ./data init platon.json
```

## 启动本地节点
- Windows命令行

```
D:\platon-node> platon.exe --identity platon --datadir .\data --port 16789 --rpcport 6789 --rpcapi "db,eth,net,web3,admin,personal" --rpc --debug --verbosity 3 --rpcaddr 0.0.0.0  --nodiscover  --syncmode "full" --gcmode "archive"
```

- Linux命令行

```
$ ./platon --identity platon --datadir ./data --port 16789 --rpcport 6789 --rpcapi "db,eth,net,web3,admin,personal" --rpc --debug --verbosity 3 --rpcaddr 0.0.0.0  --nodiscover  --syncmode "full" --gcmode "archive"
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
| --nodiscover | 不开启节点发现功能       |
| --nodekey    | 指定节点私钥文件保存路径 |

更多参数意义通过`platon --help`命令查看。

## 添加测试网络节点

1.通过`http`方式进入`platon`控制台

- Windows命令行

```
D:\platon-node> platon.exe attach http://localhost:6789
```

- Linux命令行：

```
$ ./platon attach http://localhost:6789
```

2.在控制台添加测试网络节点
```
> admin.addPeer("enode://4f2a8690b376a6fa9c23cc2ca7e9065136b55cfa1829ef037ce46717867304832c4da172e014354e96259fb4dec79f5f4c8da74bff8b03b1d86475d18143affc@13.67.44.50:16791")
```

3.在控制台查看是否添加成功
```
> admin.peers
```

4.显示如下内容则表示添加成功
```
[{
    caps: ["eth/62", "eth/63"],
    id: "4f2a8690b376a6fa9c23cc2ca7e9065136b55cfa1829ef037ce46717867304832c4da172e014354e96259fb4dec79f5f4c8da74bff8b03b1d86475d18143affc",
    name: "PlatONnetwork/Test-SEA/v0.3.0-stable-b57cf683/linux-amd64/go1.11.1",
    network: {
      consensus: false,
      inbound: false,
      localAddress: "192.168.112.71:58424",
      remoteAddress: "13.67.44.50:16791",
      static: true,
      trusted: false
    },
    protocols: {
      eth: {
        difficulty: 965297,
        head: "0x212749be417e14b8d1579654693444a939b74f9d5164821842c1574933d350ac",
        version: 63
      }
    }
}]
```
