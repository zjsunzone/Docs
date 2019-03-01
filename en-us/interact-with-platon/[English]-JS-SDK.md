Javascript SDK

- [Overview](#overview)
- [Release notes](#release-notes)
  - [v0.2.0 Release notes](#v020-release-notes)
  - [Quick start](#quick-start)
  - [Installation instruction](#installation-instruction)
  - [Code initialization](#code-initialization)
  - [Contract](#contract)
    - [Sample contract](#sample-contract)
    - [Deploy Contract](#deploy-contract)
      - [Interface declaration](#interface-declaration)
      - [Parameters](#parameters)
      - [Sample code](#sample-code)
    - [Contract function call](#contract-function-call)
      - [Interface](#interface)
      - [Parameters](#parameters-1)
      - [Sample code](#sample-code-1)
    - [Call contract sendRawTransaction](#call-contract-sendrawtransaction)
      - [Interface](#interface-1)
      - [Parameters](#parameters-2)
      - [Sample code](#sample-code-2)
    - [Built contract](#built-contract)
      - [CandidateContract](#CandidateCcontract)
      - [TicketContract](#TicketContract)
  - [web3](#web3)
  - [web3 eth related (standard JSON RPC)](#web3-eth-related-standard-json-rpc)
  - [New interfaces](#new-interfaces)
    - [contract](#contract)
      - [Interface](#interface-2)
      - [Parameters](#parameters-3)
      - [Sample code](#sample-code-3)
    - [contract.getPlatONData](#contractgetplatondata)
      - [Interface](#interface-3)
      - [Parameters](#parameters-4)
      - [Sample code](#sample-code-4)
    - [contract.decodePlatONCall](#contractdecodeplatoncall)
      - [Interface](#interface-4)
      - [Parameters](#parameters-5)
      - [Sample code](#sample-code-5)
    - [contract.decodePlatONLog](#contractdecodeplatonlog)
      - [Interface](#interface-5)
      - [Parameters](#parameters-6)
      - [Sample code](#sample-code-6)

## Overview
> Javascript SDK is a JS development kit provided for JS developers working on PlatON

## Release notes

### v0.2.0 Release notes

1. Support PlatON wasm contract

### Quick start

### Installation instruction

Installation using node.js:

`cnpm i https://github.com/PlatONnetwork/client-sdk-js`

### Code initialization

Next step is to create a web3 instance with provider. To avoid overwrite your existing provider, better to check if web3 instance already exists. For example, when use Mist.


```js
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:6789'));
}


```

### Contract

How to code wasm contract, ABI(wasm file) and BIN(json file), please refer to [Wasm Contract Development Guide](en-us/development/[English]-Wasm-Contract-Development-Guide)

#### Sample contract


```
    namespace platon {
        class ACC : public token::Token {
        public:
            ACC(){}
            void init(){
                Address user(std::string("0xa0b21d5bcc6af4dda0579174941160b9eecb6916"), true);
                initSupply(user, 199999);
            }

            void create(const char *addr, uint64_t asset){
                Address user(std::string(addr), true);
                Token::create(user, asset);
            }

            uint64_t getAsset(const char *addr) const {
                Address user(std::string(addr), true);
                return Token::getAsset(user);
            }

            void transfer(const char *addr, uint64_t asset) {
                Address user(std::string(addr), true);
                Token::transfer(user, asset);
            }
        };
    }

    PLATON_ABI(platon::ACC, create)
    PLATON_ABI(platon::ACC, getAsset)
    PLATON_ABI(platon::ACC, transfer)
    //platon autogen begin
    extern "C" {
        void create(const char * addr,unsigned long long asset) {
            platon::ACC ACC_platon;
            ACC_platon.create(addr,asset);
        }
        unsigned long long getAsset(const char * addr) {
            platon::ACC ACC_platon;
            return ACC_platon.getAsset(addr);
        }
        void transfer(const char * addr,unsigned long long asset) {
            platon::ACC ACC_platon;
            ACC_platon.transfer(addr,asset);
        }
        void init() {
            platon::ACC ACC_platon;
            ACC_platon.init();
        }
    }
    //platon autogen end


```

#### Deploy Contract

##### Interface declaration

    MyContract.deploy(deployData [, callback])

##### Parameters

|  Name   |    Type  |       Attributes  | Description |
|:-------- |:-------- |:-------- | :-------- |
|deployData |String |Mandatory | Signed transaction data in hex |
|callback |Funciton |Optional | call back function for asynchronous execution |

##### Sample code


````js
const Web3 = require('web3'),
    fs = require('fs'),
    path = require('path'),
    Tx = require('ethereumjs-tx'),
    abi = require('./multisig.cpp.abi.json') // binary interface of application in abi format
    ;

const provider = 'http://localhost:6789',
    privateKey='fd098d39e56115762cf28d7d223a4303a42a42451da6e7da37cb40a81a246d98',//Private key
    address='0xf216d6e4c17097a60ee2b8e5c88941cd9f07263b'

const web3 = new Web3(new Web3.providers.HttpProvider(provider))

const MyContract  = web3.eth.contract(abi)

const source = fs.readFileSync(path.join(__dirname, './demo.wasm'));//WebAssembly binary bin file

const platONData = MyContract.deploy.getPlatONData(source)

const params = {
    nonce: web3.eth.getTransactionCount(address),
    gas: "0x506709",
    gasPrice: "0x8250de00",
    data: platONData,
    from:address
}

const daployData=sign(privateKey,params)

let myContractReturned = MyContract.deploy(daployData, (err, myContract)=> {
    if (!err) {
        if (!myContract.address) {
            console.log("contract deploy transaction hash: " + myContract.transactionHash) // transaction hash
        } else {
            // Successful deployment
            console.log("contract deploy transaction address: " + myContract.address)// contract address deployed
            const receipt = web3.eth.getTransactionReceipt(myContract.transactionHash);
            console.log(`contract deploy receipt:`, receipt);
        }
    } else {
        console.log(`contract deploy error:`, err)
    }
});

function sign(privateKey, data) {
    const key = new Buffer(privateKey, 'hex')
    const tx = new Tx(data)
    tx.sign(key)

    const serializeTx = tx.serialize()
    const result = '0x' + serializeTx.toString('hex')
    return result
}


````

#### Contract function call

> Call contract function, return a value

##### Interface

    web3.eth.call(Object [, defaultBlock] [, callback])

##### Parameters

|Name|                  Type|         Attributes|   Description|
| :------: |:------: |:------: | :------: |
|Object |String |Mandatory| Return an object of transaction, different from `web3.eth.sendTransaction` where `from` Attributes is optional|
|defaultBlock|Number/String |Optional| use `web3.eth.defaultBlock` when empty|
|callback|Funciton  |Optional|Callback function for asynchronous execution|

##### Sample code


````js
const data = contract.getOwners.getPlatONData()

const result = web3.eth.call({
    from: web3.eth.accounts[0],
    to: contract.address,
    data: data
});

console.log('call result:', result);

contract.decodePlatONCall(result)


````

#### Call contract sendRawTransaction

> Send transaction signed b
> y private key

##### Interface

    web3.eth.sendRawTransaction(signedTransactionData [, callback])

##### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|signedTransactionData |String |Mandatory|Signed transaction data in hex|
|callback|Funciton  |Optional|Callback function for asynchronous execution|

##### Sample code


````js
const Tx = require('ethereumjs-tx');

const privateKey='4484092b68df58d639f11d59738983e2b8b81824f3c0c759edd6773f9adadfe7',
    param_from = web3.eth.accounts[0],
    param_to = wallet.address,
    param_assert = 4
    ;

const data = myContractInstance.transfer02.getPlatONData(param_from, param_to, param_assert);

const hash = web3.eth.sendRawTransaction(sign(privateKey,getParams(data)))
console.log('hash', hash);//0xb1335d4db521ddc0b390448f919e5b5af1258b29e7ab4e0d68b0ef315af0cf5f

const data=web3.eth.getTransactionReceipt(hash);

let res = myContractInstance.decodePlatONLog(data.logs[0]);

function sign(privateKey, data) {
    const key = new Buffer(privateKey, 'hex')
    const tx = new Tx(data)
    tx.sign(key)

    const serializeTx = tx.serialize()
    const result = '0x' + serializeTx.toString('hex')
    return result
}

function getParams(data = '', value = "0x0") {
    const nonce = web3.eth.getTransactionCount(wallet.address)
    value = web3.toHex(web3.toWei(value, 'ether'));

    const params = {
        from: '0xf216d6e4c17097a60ee2b8e5c88941cd9f07263b',
        gasPrice: 22 * 10e9,
        gas: 80000,
        to: myContractInstance.address,
        value,
        data,
        nonce
    }

    return params
}


````

#### Built contract

##### CandidateContract

> Candidate-related contract interface in the PlatON economic model [contract description](https://note.youdao.com/)

##### Loading contract

````js
const Web3 = require('web3'),
    wallet = require('../owner.json'),//Wallet file
    toObj=require('./toObj'),//detailed code below
    sign = require('./sign'),//Signature, detailed code below
    abi = require('../abi/candidateConstract.json'),//Candidate related abi
    getTransactionReceipt = require('./getTransactionReceipt'),//detailed code below
    candidateContractAddress='0x1000000000000000000000000000000000000001';

const web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:6789'));

const calcContract = web3.eth.contract(abi),
    candidateContract = calcContract.at(candidateContractAddress);

function getParams(data = '', value = "0x0") {
    const nonce = web3.eth.getTransactionCount(wallet.address);
    value = web3.toHex(web3.toWei(value, 'ether'));

    const params = {
        from:'0xf8f3978c14f585c920718c27853e2380d6f5db36',//wallet address
        gasPrice: 22 * 10e9,
        gas: 80000,
        to: candidateContract.address,
        value,
        data,
        nonce
    }
    return params;
}
````

./toObj.js

````js
function toObj(str) {
    if (!str || str == '0x') return null;
    let result = null;
    try {
        result = JSON.parse(str)
    } catch (error) {
        console.warn(`toObj error`, error)
        throw new Error(error)
    }
    if (Array.isArray(result)) {
        return result.map(item => {
            try {
                item.Extra ? item.Extra = JSON.parse(item.Extra) : ''
            } catch (error) {
                console.warn('Extra error',item.Extra)
            }
            return item
        })
    } else {
        try {
            result.Extra ? result.Extra = JSON.parse(result.Extra) : ''
        } catch (error) {
            console.warn('Extra error',result.Extra)
        }
        return result
    }
    return result;
}

module.exports = toObj;
````

./sign.js file

````js
const Tx = require('ethereumjs-tx');

module.exports = (privateKey, data) => {
    if (!privateKey || !data) {
        throw new Error(`Signature parameter exception`);
    }

    const key = new Buffer(privateKey, 'hex'),
        tx = new Tx(data);

    tx.sign(key);

    const serializeTx = tx.serialize(),
        result = '0x' + serializeTx.toString('hex');

    return result;
};
````

./getTransactionReceipt.js

````js
const Web3 = require('web3'),
    config = require('../config/config.json');
const web3 = new Web3 (new Web3.providers.HttpProvider (config.provider));

let wrapCount = 60;
function getTransactionReceipt(hash, fn) {
    let id = '',
        result = web3.eth.getTransactionReceipt(hash),
        data = {};
    if (result && result.transactionHash && hash == result.transactionHash) {
        clearTimeout(id);
        if (result.logs.length != 0) {
            fn(0, result);
        } else {
            fn(1001, 'contract exception, failure');
        }
    } else {
        if (wrapCount--) {
            id = setTimeout(() => {
                getTransactionReceipt(hash, fn);
            }, 1000);
        } else {
            fn(1000, 'time out');
            id = '';
        }
    }
}

module.exports = getTransactionReceipt
````

##### CandidateDeposit

> Node candidate application / increase pledge

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|nodeId|String|Mandatory|Node id, `16` format, starting with `0x`|
|owner|String|Mandatory|Qualifying deposit refund address, `16` format, starting with `0x`|
|fee|String|Mandatory|Block rewards commission ratio, based on 10000 (eg: 5%, then fee=500)|
|host|String|Mandatory|Node IP|
|port|String|Mandatory|Node P2P port number|
|Extra|String|Mandatory|Additional data|

Extra描述

```
{
    "nodeName":string,                     //Node name
    "officialWebsite":string,              //official website http | https
    "nodePortrait":string,                 //node logo http | https
    "nodeDiscription":string,              //Introduction
    "nodeDepartment":string                //organization name
}
```

**Return value or callback**

| Name |Type|Description|
| :------: |:------: |:------: |
|param1|String|parsed log array|

param1 description
```
{
	"Ret":boolean,                         //whether succeed. True: successful false: failed
	"ErrMsg":string                        //Error message, there is a failure
}
```

###### Sample code

````js
const privateKey = '099cad12189e848f70570196df434717c1ccc04f421da6ab651f38297a065cb7';

const nodeId = '0xeebeaa496d954f8ee864e6460719755398f1e5b36e7a0c911f527fe3247b02a0a4db17aa59c5235e923602df1aeb26042149b8d2fd71cf990046b08d3b323b9a', //[64]byte node ID (public key)
    owner = '0xf8f3978c14f585c920718c27853e2380d6f5db36', //[20]byte deposit refund address
    fee = 500, //Uint32 block reward commission ratio, based on 10000 (eg: 5%, then fee = 500)
    { host, port } = config, //String: node IP string: node port number
    Extra = web3.toHex({
        nodeName: 'node name',
        nodeDiscription: 'node discription',
        nodeDepartment: 'institution name',
        officialWebsite: 'https://www.platon.network',
        nodePortrait: 'http://192.168.9.86:8082/group2/M00/00/00/wKgJVlr0KDyAGSddAAYKKe2rswE261.png',
        time: Date.now(), //join time
        //nodeId,//node ID
        owner, //node refund address
    }), //string,additional data (with length limit, limit value to be determined)
    value = 5;//quality deposit

Extra = web3.toUnicode(Extra);

const data = candidateContract.CandidateDeposit.getPlatONData(nodeId, owner, fee, host, port, Extra, {
    transactionType:1001
});

const hash = web3.eth.sendRawTransaction(sign(wallet, password, getParams(data, value)))

getTransactionReceipt(hash, (code, data) => {
    if (code == 0) {
        let res = candidateContract.decodePlatONLog(data.logs[0]);
        if (res.length && res[0]) {
            res = JSON.parse(res[0])
            if (res.ErrMsg == 'success') {
                console.log(`success`);
            } else {
                console.log(`failure`);
            }
        } else {
            console.log(`failure`)
        }
    } else {
        console.warn(`abnormal`);
    }
});
````

##### CandidateApplyWithdraw

> The node quality deposit refund application, the node will be reordered after the application is successful, and the initiated address must be the address of the quality deposit refund. from==owner

###### Parameters

|Name|Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|nodeId|String|Mandatory|node id, hexadecimal format, starting with 0x|
|withdraw|String|Mandatory|refund amount (unit: wei)|

**Return value or callback**

| Name |Type|Description|
| :------: |:------: |:------: |
|param1|String|parsed log array|

param1 description
```
{
	"Ret":boolean,                         //whether succeed. True: successful false: failed
	"ErrMsg":string                        //Error message, there is a failure
}
```

###### Sample code

````js
const privateKey = '099cad12189e848f70570196df434717c1ccc04f421da6ab651f38297a065cb7';
const nodeId = '0xeebeaa496d954f8ee864e6460719755398f1e5b36e7a0c911f527fe3247b02a0a4db17aa59c5235e923602df1aeb26042149b8d2fd71cf990046b08d3b323b9a', //[64]byte node ID (public key)
    withdraw = Number(web3.toWei(5, 'ether'));
const data = candidateContract.CandidateApplyWithdraw.getPlatONData(nodeId, withdraw, {
    transactionType:1002
});
const hash = web3.eth.sendRawTransaction(sign(privateKey, getParams(data)))

getTransactionReceipt(hash, (code, data) => {
    console.log(code, data)
    if (code == 0) {
        let res = candidateContract.decodePlatONLog(data.logs[0], 'CandidateApplyWithdrawEvent');
        if (res.length && res[0]) {
            res = JSON.parse(res[0])
            if (res.ErrMsg == 'success') {
                console.log(`success`);
            } else {
                console.log(`failure`);
            }
        } else {
            console.log(`failure`)
        }
    } else {
        console.warn(`abnormal`)
    }
})
````

##### CandidateWithdraw

> The node quality deposit is extracted. After the call is successful, all the refunded quality deposits will be extracted to the owner account.

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|nodeId|String|Mandatory|node id, hexadecimal format, starting with 0x|

**Return value or callback**

| Name |Type|Description|
| :------: |:------: |:------: |
|param1|String|parsed log array|

param1 description
```
{
	"Ret":boolean,                         //whether succeed. True: successful false: failed
	"ErrMsg":string                        //Error message, there is a failure
}
```

###### Sample code

````js
const privateKey = '099cad12189e848f70570196df434717c1ccc04f421da6ab651f38297a065cb7';
const nodeId = '0xeebeaa496d954f8ee864e6460719755398f1e5b36e7a0c911f527fe3247b02a0a4db17aa59c5235e923602df1aeb26042149b8d2fd71cf990046b08d3b323b9a' //[64]byte node ID (public key)
const data = candidateContract.CandidateWithdraw.getPlatONData(nodeId, {
    transactionType:1003
});
const hash = web3.eth.sendRawTransaction(sign(privateKey, getParams(data)))
console.log('hash', hash)
getTransactionReceipt(hash, (code, data) => {
    console.log(code, data)
    if (code == 0) {
        let res = candidateContract.decodePlatONLog(data.logs[0], 'CandidateWithdrawEvent');
        if (res.length && res[0]) {
            res = JSON.parse(res[0])
            if (res.ErrMsg == 'success') {
                console.log(`success`);
            } else {
                console.log(`failure`);
            }
        } else {
            console.log(`failure`)
        }
    } else {
        console.warn(`abnormal`);
    }
})
````

##### SetCandidateExtra

> Set node additional information, the originated address must be the address of the quality deposit refund from==owner

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|Extra|String|Mandatory|additional data, hex string|

Extra描述

```
{
    "nodeName":string,                     //node name
    "officialWebsite":string,              //official website http | https
     "nodePortrait":string,                 //node logo http | https
    "nodeDiscription":string,              //brief Introduction
    "nodeDepartment":string                //institution name
}
```
**Return value or callback**

|Name|Type|Description|
| :------: |:------: |:------: |
|param1|String|parsed log array|

param1 description
```
{
	"Ret":boolean,                         //whether succeed. True: successful false: failed
	"ErrMsg":string                        //Error message, there is a failure
}
```

###### Sample code

````js
const privateKey = '099cad12189e848f70570196df434717c1ccc04f421da6ab651f38297a065cb7';
let nodeId = '0xeebeaa496d954f8ee864e6460719755398f1e5b36e7a0c911f527fe3247b02a0a4db17aa59c5235e923602df1aeb26042149b8d2fd71cf990046b08d3b323b9a', //[64]byte node ID (public key)
    owner = wallet.address, //[20]byte deposit refund address
    time = Date.now(),
    Extra =JSON.stringify({
        nodeName: 'node name',
        nodeDiscription: 'node introduction' + time,
        nodeDepartment: 'institution name' + time,
        officialWebsite: 'www.platon.network',
        nodePortrait: 'URL',
        time: time, //join time
        owner, //node refund address
    }) //string,additional data (with length limit, limit value to be determined)

Extra = web3.toUnicode(Extra);

const data = candidateContract.SetCandidateExtra.getPlatONData(nodeId, Extra, {
    transactionType:1004
});
const hash = web3.eth.sendRawTransaction(sign(privateKey, getParams(data)))

getTransactionReceipt(hash, (code, data) => {
    console.log(code, data)
    let res = candidateContract.decodePlatONLog(data.logs[0]);
    if (res && res[0]) {
        res = JSON.parse(res[0])
        if (res.ErrMsg == 'success') {
            console.log(`success`);
        } else {
            console.error(`failure`);
        }
    } else {
        console.error(`failure`)
    }
})
````

##### CandidateWithdrawInfos

> Get a list of refund records requested by the node

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|nodeId|String|Mandatory|node id, hexadecimal format, starting with 0x|

**Return value or callback**

| Name |Type|Description|
| :------: |:------: |:------: |
|string|String|parsed log array|

```
{
    "Ret": true,
    "ErrMsg": "success",
    "Infos": [{                        //refund record
        "Balance": 100,                //refund amount
        "LockNumber": 13112,           //refund application block height
        "LockBlockCycle": 1            //refund amount lock cycle
    }]
}
```

###### Sample code

````js
const nodeId = '0xeebeaa496d954f8ee864e6460719755398f1e5b36e7a0c911f527fe3247b02a0a4db17aa59c5235e923602df1aeb26042149b8d2fd71cf990046b08d3b323b9a' //[64]byte node ID (public key)
const data = candidateContract.CandidateWithdrawInfos.getPlatONData(nodeId)

const result = web3.eth.call({
    from: wallet.address,
    to: candidateContract.address,
    data: data,
});

const result1 = candidateContract.decodePlatONCall(result);
const result2 = toObj(result1.data)
console.log(result2);
````

##### CandidateDetails

> Get candidate information

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|nodeId|String|Mandatory|node id, hexadecimal format, starting with 0x|

```
{
    //Quality deposit
    "Deposit": 200,
    //The latest block height of the quality deposit update
    "BlockNumber": 12206,
    //Block trading index
    "TxIndex": 0,
    //Node Id
    "CandidateId": "6bad331aa2ec6096b2b6034570e1761d687575b38c3afc3a3b5f892dac4c86d0fc59ead0f0933ae041c0b6b43a7261f1529bad5189be4fba343875548dc9efd3",
    //Node IP
    "Host": "192.168.9.76",
    //Node P2P port number
    "Port": "26794",
    //Quality deposit refund address
    "Owner": "0xf8f3978c14f585c920718c27853e2380d6f5db36",
    //The sender of the latest pledge transaction
    "From": "0x493301712671ada506ba6ca7891f436d29185821",
    //Additional data
    "Extra": "{\"nodeName\":\"xxxx-noedeName\",\"officialWebsite\":\"xxxx-officialWebsite\",\"nodePortrait\":\"group2/M00/00/12/wKgJVlw0XSyAY78cAAH3BKJzz9Y83.jpeg\",\"nodeDiscription\":\"xxxx-nodeDiscription1\",\"nodeDepartment\":\"xxxx-nodeDepartment\"}",
    //Block rewards commission ratio, based on 10000 (eg: 5%, then fee=500)
    "Fee": 500
}
```
**Return value or callback**

| Name |Type|Description|
| :------: |:------: |:------: |
|param1|String|parsed log array|

param1 description
```
{
	"Ret":boolean,                         //whether succeed. True: successful false: failed
	"ErrMsg":string                        //Error message, there is a failure
}
```

###### Sample code

````js
const nodeId = '0xeebeaa496d954f8ee864e6460719755398f1e5b36e7a0c911f527fe3247b02a0a4db17aa59c5235e923602df1aeb26042149b8d2fd71cf990046b08d3b323b9a' // Node ID (public key)

const data = candidateContract.CandidateDetails.getPlatONData(nodeId)

const result = web3.eth.call({
    from: wallet.address,
    to: candidateContract.address,
    data: data,
});

const result1 = candidateContract.decodePlatONCall(result);
const result2 = toObj(result1.data)
console.log(result2);
````

##### candidateDetails

> Get candidate information

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|nodeId|String|Mandatory|node id, hexadecimal format, starting with 0x|

**Return value or callback**

`string` - `String` Json format string

```
{
    //Quality deposit
    "Deposit": 200,
    //The latest block height of the quality deposit update
    "BlockNumber": 12206,
    //Block trading index
    "TxIndex": 0,
    //Node Id
    "CandidateId": "6bad331aa2ec6096b2b6034570e1761d687575b38c3afc3a3b5f892dac4c86d0fc59ead0f0933ae041c0b6b43a7261f1529bad5189be4fba343875548dc9efd3",
    //Node IP
    "Host": "192.168.9.76",
    //Node P2P port number
    "Port": "26794",
    //Quality deposit refund address
    "Owner": "0xf8f3978c14f585c920718c27853e2380d6f5db36",
    //The sender of the latest pledge transaction
    "From": "0x493301712671ada506ba6ca7891f436d29185821",
    //Additional data
    "Extra": "{\"nodeName\":\"xxxx-noedeName\",\"officialWebsite\":\"xxxx-officialWebsite\",\"nodePortrait\":\"group2/M00/00/12/wKgJVlw0XSyAY78cAAH3BKJzz9Y83.jpeg\",\"nodeDiscription\":\"xxxx-nodeDiscription1\",\"nodeDepartment\":\"xxxx-nodeDepartment\"}",
    //Block rewards commission ratio, based on 10000 (eg: 5%, then fee=500)
    "Fee": 500
}
```

###### Sample code

````js
const nodeId = '0xeebeaa496d954f8ee864e6460719755398f1e5b36e7a0c911f527fe3247b02a0a4db17aa59c5235e923602df1aeb26042149b8d2fd71cf990046b08d3b323b9a' // Node ID (public key)

const data = candidateContract.CandidateDetails.getPlatONData(nodeId)

const result = web3.eth.call({
    from: wallet.address,
    to: candidateContract.address,
    data: data,
});

const result1 = candidateContract.decodePlatONCall(result);
const result2 = toObj(result1.data)
console.log(result2);
````

##### CandidateList

> Get a list of all the finalist information

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |

**Return value or callback**

`string` - `String` Json format string

```
[{
    "Deposit": 11100000000000000000,
    "BlockNumber": 13721,
    "TxIndex": 0,
    "CandidateId": "c0e69057ec222ab257f68ca79d0e74fdb720261bcdbdfa83502d509a5ad032b29d57c6273f1c62f51d689644b4d446064a7c8279ff9abd01fa846a3555395535",
    "Host": "192.168.9.76",
    "Port": "26793",
    "Owner": "0x3ef573e439071c87fe54287f07fe1fd8614f134c",
    "From": "0x3ef573e439071c87fe54287f07fe1fd8614f134c",
    "Extra": "{\"nodeName\":\"xxxx-noedeName\",\"officialWebsite\":\"xxxx-officialWebsite\",\"nodePortrait\":\"group2/M00/00/12/wKgJVlw0XSyAY78cAAH3BKJzz9Y83.jpeg\",\"nodeDiscription\":\"xxxx-nodeDiscription1\",\"nodeDepartment\":\"xxxx-nodeDepartment\"}",
    "Fee": 9900
}, {
    "Deposit": 200,
    "BlockNumber": 12206,
    "TxIndex": 0,
    "CandidateId": "6bad331aa2ec6096b2b6034570e1761d687575b38c3afc3a3b5f892dac4c86d0fc59ead0f0933ae041c0b6b43a7261f1529bad5189be4fba343875548dc9efd3",
    "Host": "192.168.9.76",
    "Port": "26794",
    "Owner": "0xf8f3978c14f585c920718c27853e2380d6f5db36",
    "From": "0x493301712671ada506ba6ca7891f436d29185821",
    "Extra": "{\"nodeName\":\"xxxx-noedeName\",\"officialWebsite\":\"xxxx-officialWebsite\",\"nodePortrait\":\"group2/M00/00/12/wKgJVlw0XSyAY78cAAH3BKJzz9Y83.jpeg\",\"nodeDiscription\":\"xxxx-nodeDiscription1\",\"nodeDepartment\":\"xxxx-nodeDepartment\"}",
    "Fee": 500
}]
```

###### Sample code

````js
const data = candidateContract.CandidateList.getPlatONData()

const result = web3.eth.call({
    from: wallet.address,
    to: candidateContract.address,
    data: data,
});

const result1 = candidateContract.decodePlatONCall(result);
const result2 = toObj(result1.data)
console.log(result2);
````

##### VerifiersList

> Get a list of certifiers participating in the current consensus

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |

**Return value or callback**

`string` - `String` Json format string

```
[{
    "Deposit": 11100000000000000000,
    "BlockNumber": 13721,
    "TxIndex": 0,
    "CandidateId": "c0e69057ec222ab257f68ca79d0e74fdb720261bcdbdfa83502d509a5ad032b29d57c6273f1c62f51d689644b4d446064a7c8279ff9abd01fa846a3555395535",
    "Host": "192.168.9.76",
    "Port": "26793",
    "Owner": "0x3ef573e439071c87fe54287f07fe1fd8614f134c",
    "From": "0x3ef573e439071c87fe54287f07fe1fd8614f134c",
    "Extra": "{\"nodeName\":\"xxxx-noedeName\",\"officialWebsite\":\"xxxx-officialWebsite\",\"nodePortrait\":\"group2/M00/00/12/wKgJVlw0XSyAY78cAAH3BKJzz9Y83.jpeg\",\"nodeDiscription\":\"xxxx-nodeDiscription1\",\"nodeDepartment\":\"xxxx-nodeDepartment\"}",
    "Fee": 9900
}, {
    "Deposit": 200,
    "BlockNumber": 12206,
    "TxIndex": 0,
    "CandidateId": "6bad331aa2ec6096b2b6034570e1761d687575b38c3afc3a3b5f892dac4c86d0fc59ead0f0933ae041c0b6b43a7261f1529bad5189be4fba343875548dc9efd3",
    "Host": "192.168.9.76",
    "Port": "26794",
    "Owner": "0xf8f3978c14f585c920718c27853e2380d6f5db36",
    "From": "0x493301712671ada506ba6ca7891f436d29185821",
    "Extra": "{\"nodeName\":\"xxxx-noedeName\",\"officialWebsite\":\"xxxx-officialWebsite\",\"nodePortrait\":\"group2/M00/00/12/wKgJVlw0XSyAY78cAAH3BKJzz9Y83.jpeg\",\"nodeDiscription\":\"xxxx-nodeDiscription1\",\"nodeDepartment\":\"xxxx-nodeDepartment\"}",
    "Fee": 500
}]
```

###### Sample code

````js
const data = candidateContract.VerifiersList.getPlatONData()

const result = web3.eth.call({
    from: wallet.address,
    to: candidateContract.address,
    data: data,
});

const result1 = candidateContract.decodePlatONCall(result);
const result2 = toObj(result1.data);
console.log(result2);
````







##### TicketContract

> Ticket pool related contract interface in PlatON economic model [contract description](https://note.youdao.com/)

##### Loading contract

````js
const Web3 = require('web3'),
    wallet = require('../owner.json'),//wallet file
    sign = require('./sign'),//Signature, detailed code below
    abi = require('../abi/ticketContract.json'),//ticket pool related abi
    getTransactionReceipt = require('./getTransactionReceipt'),//detailed code below
    ticketContractAddress='0x1000000000000000000000000000000000000002';//ticket pool contract address

const web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:6789'));

const calcContract = web3.eth.contract(abi),
    ticketContract = calcContract.at(ticketContractAddress);

function getParams(data = '', value = "0x0") {
    const nonce = web3.eth.getTransactionCount(wallet.address);

    value = web3.toHex(value)

    const params = {
        from:'0xf8f3978c14f585c920718c27853e2380d6f5db36',//wallet address
        gasPrice: 22 * 10e9,
        gas: 80000,
        to: ticketContract.address,
        value,
        data,
        nonce
    }
    return params;
}
````

./sign.js file

````js
const Tx = require('ethereumjs-tx');

module.exports = (privateKey, data) => {
    if (!privateKey || !data) {
        throw new Error(`Signature parameter exception`);
    }

    const key = new Buffer(privateKey, 'hex'),
        tx = new Tx(data);

    tx.sign(key);

    const serializeTx = tx.serialize(),
        result = '0x' + serializeTx.toString('hex');

    return result;
};
````

./getTransactionReceipt.js

````js
const Web3 = require('web3'),
    config = require('../config/config.json');
const web3 = new Web3 (new Web3.providers.HttpProvider (config.provider));

let wrapCount = 60;
function getTransactionReceipt(hash, fn) {
    let id = '',
        result = web3.eth.getTransactionReceipt(hash),
        data = {};
    if (result && result.transactionHash && hash == result.transactionHash) {
        clearTimeout(id);
        if (result.logs.length != 0) {
            fn(0, result);
        } else {
            fn(1001, 'contract exception, failure');
        }
    } else {
        if (wrapCount--) {
            id = setTimeout(() => {
                getTransactionReceipt(hash, fn);
            }, 1000);
        } else {
            fn(1000, 'time out');
            id = '';
        }
    }
}

module.exports = getTransactionReceipt
````

##### GetTicketPrice

> Get the current fare in units of `wei`

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |

**Return value or callback**

| Name |Type|Description|
| :------: |:------: |:------: |
|param1|String|parsed log array|

param1 description
```
{
	"Ret":Int,                         //Current fare (in e)
	"ErrMsg":string                    //Error message, there is a failure
}
```

###### Sample code

````js
const data = ticketContract.GetTicketPrice.getPlatONData()

const result = web3.eth.call({
    from: wallet.address,
    to: ticketContract.address,
    data: data,
});

const result1 = ticketContract.decodePlatONCall(result);
console.log('getTicketPrice result:', result1);
````

##### VoteTicket

> Buy a ballot and vote for the (existing) candidate. The ticket pool capacity is 51200. The value of the sent transaction is the number of tickets purchased * The unit price of the ballot

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|count|Number|Mandatory|Number of tickets purchased|
|price|Number|Mandatory|The unit price of the ballot (in units of `e`)|
|nodeId|String|Mandatory|Node id, `16` format, starting with `0x`|


**Return value or callback**

| Name |Type|Description|
| :------: |:------: |:------: |
|param1|String|parsed log array|

param1 description
```
{
	"Ret":boolean,                         //whether succeed. True: successful false: failed
	"ErrMsg":string                        //Error message, there is a failure
    "Data":string                          //Return data (number of successful votes)
}
```

###### Sample code

````js
const jsSHA = require('jssha');
const privateKey = '099cad12189e848f70570196df434717c1ccc04f421da6ab651f38297a065cb7';

const
    count = 10,//number of tickets purchased
    price = 1,//ballot unit price Use the GetTicketPrice interface to query the current fare
    nodeId = '0x4f6c8fd10bfb512793f81a3594120c76b6991d3d06c0cc652035cbfae3fcd7cdc3f3d7a82021dfdb9ea99f014755ec1a640d832a0362b47be688bb31d504f62d'//candidate node ID

const data = ticketContract.VoteTicket.getPlatONData(count, price, nodeId, {
    transactionType: 1000//transaction type
});

const value = price * count

const hash = web3.eth.sendRawTransaction(
    sign(password, getParams(data, value))
);

getTransactionReceipt(hash, (code, data) => {
    let res = ticketContract.decodePlatONLog(data.logs[0]);
    if (res.length && res[0]) {
        res = JSON.parse(res[0]);
        if (res.ErrMsg == 'success') {
            if (res.Data) {
                let num = res.Data - 0, ids = [];
                for (let i = 0; i < num; i++) {
                    let ticketIndex = (i + '').charCodeAt(0).toString(16)
                    let str = txHash.replace(/^0x/, '') + ticketIndex;
                    var shaObj = new jsSHA("SHA3-256", "HEX");
                    shaObj.update(str);
                    let id = '0x' + shaObj.getHash("HEX");
                    ids.push(id);
                }
                console.log(ids)
            } else {
                console.warn('No ticket id')
            }
        } else {
            console.warn(`Purchase of ballot failed`);
        }
    } else {
        console.warn(`Purchase of ballot failed`);
    }
});
````

##### GetTicketDetail

> Get ticket details

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|ticketId|String|Mandatory|Ticket Id, `16` format, starting with `0x`|

**Return value or callback**

| Name |Type|Description|
| :------: |:------: |:------: |
|param1|String|parsed log array|

param1 description
```
{
	"Ret":Ticket,                      //Ballot information
	"ErrMsg":string                    //Error message, there is a failure
}
```

###### Sample code

````js
const ticketId = "0x134fba852817b9da8508f4b7e82e792be05b90f2a288e52df17c10da0f303b65"//票Id

const data = ticketContract.GetTicketDetail.getPlatONData(ticketId)

const result = web3.eth.call({
    from: wallet.address,
    to: ticketContract.address,
    data: data,
});

const result1 = ticketContract.decodePlatONCall(result);
console.log('getBatchTicketDetail:', result1);
````

##### GetBatchTicketDetail

> Get ticket details in bulk

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|ticketIds|String|Mandatory|List of node ids, `16` format, starting with `0x`, string of multiple node ids spliced by ":"|

**Return value or callback**

| Name |Type|Description|
| :------: |:------: |:------: |
|param1|String|parsed log array|

param1 description
```
{
	"Ret":[]Ticket,                    //Ballot information list
	"ErrMsg":string                    //Error message, there is a failure
}
```

###### Sample code

````js
const ticketIds = "0x134fba852817b9da8508f4b7e82e792be05b90f2a288e52df17c10da0f303b65:0x9d1078cb595b669dc37501c4a6ed5bf98732d15ec083f4ad102b677ce62d07dc:0x036809aaa312a4414ffc0bfe9cdd1dadd9fd54725e1d8305fea8b39a566506e5:0xb0144ebd80ee817902185da65c23daa16eeceb339125ce889841925e928963ad:0x80d8bab01789f512d9d8b060609009276bf0a6b101b19989c3946e51049708fb:0x861c9a791df9d03b54471f7fd21c9e996cbaf6f6f885e47f1a20f204156ada88:0x294b2baae5f9445363436ff2cffaeff63baf536c5d21fd17b25ba0f79c30aacb:0xc8d43bf85d4a9c63198439a6c282a0b308cbb0e2102493c34640afc998f3a1ef"//ticket Id list, multiple tickets Id pass: splicing string

const data = ticketContract.GetBatchTicketDetail.getPlatONData(ticketIds)

const result = web3.eth.call({
    from: wallet.address,
    to: ticketContract.address,
    data: data,
});

const result1 = ticketContract.decodePlatONCall(result);
console.log('getBatchTicketDetail:', result1);
````

##### GetCandidateTicketIds

> Get a list of the votes of the specified candidate Id

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|nodeId|String|Mandatory|Node id, `16` format, starting with `0x`|

**Return value or callback**

| Name |Type|Description|
| :------: |:------: |:------: |
|param1|String|parsed log array|

param1 description
```
{
	"Ret":[]ticketId,                  //list of votes for Id
	"ErrMsg":string                    //Error message, there is a failure
}
```

###### Sample code

````js
const nodeId = '0x4f6c8fd10bfb512793f81a3594120c76b6991d3d06c0cc652035cbfae3fcd7cdc3f3d7a82021dfdb9ea99f014755ec1a640d832a0362b47be688bb31d504f62d'//Candidate node ID

const data = ticketContract.GetCandidateTicketIds.getPlatONData(nodeId)

const result = web3.eth.call({
    from: wallet.address,
    to: ticketContract.address,
    data: data,
});

const result1 = ticketContract.decodePlatONCall(result);
console.log('getCandidateEpoch result:', result1);
````

##### GetBatchCandidateTicketIds

> 批量获取指定候选人的选票Id的列表

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|nodeIds|String|Mandatory|节点id列表, `16`进制格式， `0x`开头，多个节点id通过":"拼接的字符串|

**Return value or callback**

| Name |Type|Description|
| :------: |:------: |:------: |
|param1|String|parsed log array|

param1 description
```
{
	"Ret":Int,                         //map(nodeId)[]ticketId 多个节点的选票的Id列表(map)
	"ErrMsg":string                    //Error message, there is a failure
}
```

###### Sample code

````js
const nodeIds = '0x4f6c8fd10bfb512793f81a3594120c76b6991d3d06c0cc652035cbfae3fcd7cdc3f3d7a82021dfdb9ea99f014755ec1a640d832a0362b47be688bb31d504f62d:0x01d033b5b07407e377a3eb268bdc3f07033774fb845b7826a6b741430c5e6b719bda5c4877514e8052fa5dbc2f20fb111a576f6696b6a16ca765de49e11e0541'//Multiple nodeId strings spliced by ":"

const data = ticketContract.GetBatchCandidateTicketIds.getPlatONData(nodeIds)

const result = web3.eth.call({
    from: wallet.address,
    to: ticketContract.address,
    data: data,
});

const result1 = ticketContract.decodePlatONCall(result);
console.log('getBatchCandidateTicketIds result:', result1);
````

##### GetCandidateEpoch

> Get the age of the designated candidate

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|nodeId|String|Mandatory|Node id, `16` format, starting with `0x`|

**Return value or callback**

| Name |Type|Description|
| :------: |:------: |:------: |
|param1|String|parsed log array|

param1 description
```
{
	"Ret":number,                      //ticket age (if no query returns 0)
	"ErrMsg":string                    //Error message, there is a failure
}
```

###### Sample code

````js
const nodeId = '0x4f6c8fd10bfb512793f81a3594120c76b6991d3d06c0cc652035cbfae3fcd7cdc3f3d7a82021dfdb9ea99f014755ec1a640d832a0362b47be688bb31d504f62d'//Candidate node ID

const data = ticketContract.GetCandidateEpoch.getPlatONData(nodeId)

const result = web3.eth.call({
    from: wallet.address,
    to: ticketContract.address,
    data: data,
});

const result1 = ticketContract.decodePlatONCall(result);
console.log('getCandidateEpoch result:', result1);
````

##### GetPoolRemainder

> Get the number of remaining votes in the ticket pool

###### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |

**Return value or callback**

| Name |Type|Description|
| :------: |:------: |:------: |
|param1|String|parsed log array|

param1 description
```
{
	"Ret":number,                      //Remaining votes
	"ErrMsg":string                    //Error message, there is a failure
}
```

###### Sample code

````js
const data = ticketContract.GetPoolRemainder.getPlatONData()

const result = web3.eth.call({
    from: wallet.address,
    to: ticketContract.address,
    data: data,
});

const result1 = ticketContract.decodePlatONCall(result);
console.log('getPoolRemainder result:', result1);
````

### web3

----

### web3 eth related (standard JSON RPC)
- For detailed api usage please refer [web3j github](https://github.com/ethereum/wiki/wiki/JavaScript-API)

### New interfaces

#### contract

##### Interface

    web3.eth.contract(abi)

##### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|abi|Array|Mandatory|Binary interface of application like ABI file|

**Return value or callback**

`Object` - a contract object

##### Sample code


````js
const abi=[
    {
        "name": "initWallet",
        "inputs": [
            {
                "name": "owner",
                "type": "string"
            },{
                "name": "required",
                "type": "uint64"
            }
        ],
        "outputs": [],
        "constant": "false",
        "type": "function"
    },{
        "name": "submitTransaction",
        "inputs": [
            {
                "name": "destination",
                "type": "string"
            },{
                "name": "from",
                "type": "string"
            },{
                "name": "value",
                "type": "uint64"
            },{
                "name": "data",
                "type": "string"
            },{
                "name": "len",
                "type": "uint64"
            },{
                "name": "time",
                "type": "uint64"
            },{
                "name": "fee",
                "type": "uint64"
            }
        ],
        "outputs": [
            {
                "name": "",
                "type": "uint64"
            }
        ],
        "constant": "false",
        "type": "function"
    },{
        "name": "confirmTransaction",
        "inputs": [
            {
                "name": "transactionId",
                "type": "uint64"
            }
        ],
        "outputs": [],
        "constant": "false",
        "type": "function"
    },{
        "name": "revokeConfirmation",
        "inputs": [
            {
                "name": "transactionId",
                "type": "uint64"
            }
        ],
        "outputs": [],
        "constant": "false",
        "type": "function"
    },{
        "name": "executeTransaction",
        "inputs": [
            {
                "name": "transactionId",
                "type": "uint64"
            }
        ],
        "outputs": [],
        "constant": "false",
        "type": "function"
    },{
        "name": "isConfirmed",
        "inputs": [
            {
                "name": "transactionId",
                "type": "uint64"
            }
        ],
        "outputs": [
            {
                "name": "",
                "type": "int32"
            }
        ],
        "constant": "true",
        "type": "function"
    },{
        "name": "getRequired",
        "inputs": [],
        "outputs": [
            {
                "name": "",
                "type": "uint64"
            }
        ],
        "constant": "true",
        "type": "function"
    },{
        "name": "getConfirmationCount",
        "inputs": [
            {
                "name": "transactionId",
                "type": "uint64"
            }
        ],
        "outputs": [
            {
                "name": "",
                "type": "uint64"
            }
        ],
        "constant": "true",
        "type": "function"
    },{
        "name": "getTransactionCount",
        "inputs": [
            {
                "name": "pending",
                "type": "int32"
            },{
                "name": "executed",
                "type": "int32"
            }
        ],
        "outputs": [
            {
                "name": "",
                "type": "uint64"
            }
        ],
        "constant": "true",
        "type": "function"
    },{
        "name": "getTransactionList",
        "inputs": [
            {
                "name": "from",
                "type": "uint64"
            },{
                "name": "to",
                "type": "uint64"
            }
        ],
        "outputs": [
            {
                "name": "",
                "type": "string"
            }
        ],
        "constant": "true",
        "type": "function"
    },{
        "name": "getOwners",
        "inputs": [],
        "outputs": [
            {
                "name": "",
                "type": "string"
            }
        ],
        "constant": "true",
        "type": "function"
    },{
        "name": "getConfirmations",
        "inputs": [
            {
                "name": "transactionId",
                "type": "uint64"
            }
        ],
        "outputs": [
            {
                "name": "",
                "type": "string"
            }
        ],
        "constant": "true",
        "type": "function"
    },
    {
        "name": "getTransactionIds",
        "inputs": [
            {
                "name": "from",
                "type": "uint64"
            },{
                "name": "to",
                "type": "uint64"
            },{
                "name": "pending",
                "type": "int32"
            },{
                "name": "executed",
                "type": "int32"
            }
        ],
        "outputs": [
            {
                "name": "",
                "type": "string"
            }
        ],
        "constant": "true",
        "type": "function"
    }
]

const MyContract = web3.eth.contract(abi);

const myContractInstance = MyContract.at('0x91b0ac240b62de2f0152cac322c6c5eafe730a84');


````


````js
var MyContract = web3.eth.contract(abi);

// instantiate by address
var contractInstance = MyContract.at([address]);

// deploy new contract
var contractInstance = MyContract.new([contructorParam1] [, contructorParam2], {data: '0x12345...', from: myAccount, gas: 1000000});

// Get the data to deploy the contract manually
var contractData = MyContract.new.getData([contructorParam1] [, contructorParam2], {data: '0x12345...'});
// contractData = '0x12345643213456000000000023434234'


````

#### contract.getPlatONData

##### Interface

    MyContract.myMethod.getPlatONData(param1 [, param2, ...])

##### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|param1/param2/...|String/Number|Mandatory|zero or more parameters|

**Return value or callback**

`Object` - one contract object

##### Sample code

#### contract.decodePlatONCall

> Parse call result based on ABI file details, like outputType defined by fnName in ABI

##### Interface

    MyContract.decodePlatONCall( callResult, [fnName])

##### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|callResult|String|Mandatory|Binary interface of application, like ABI file|
|fnName|String|Optional| interpret callResult as string when missing, otherwise, will parse based on details defined in abi file|

**Return value or callback**

`code` - 0 Success
`data` - Parsed data

##### Sample code


````js
    MyContract.decodePlatONCall( '0x',)


````

#### contract.decodePlatONLog

> Parsing a log from transaction receipt logs

##### Interface

    MyContract.decodePlatONLog(log)

##### Parameters

| Name |Type|Attributes|Description|
| :------: |:------: |:------: | :------: |
|log||Object| a log from transaction receipt logs|

**Return value or callback**

`Array` - Parsed array of logs

##### Sample code


````js
const data=web3.eth.getTransactionReceipt('0xb1335d4db521ddc0b390448f919e5b5af1258b29e7ab4e0d68b0ef315af0cf5f');

let res = myContractInstance.decodePlatONLog(data.logs[0]);


````
