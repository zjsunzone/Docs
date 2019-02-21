[TOC]

## 概述

本文档主要提供隐私合约、隐私数据服务开发包、隐私客户端SDK的详细使用说明，帮助用户深入理解隐私合约开发。

## 隐私合约

**隐私合约**是 [PlatON](zh-cn/basics/[Chinese-Simplified]-快速指南) 上一种特殊的 MPC 合约，使用`C++`语言进行开发，运行在 PlatON 节点的MPC计算虚拟机（MPC VM）里。隐私合约借助 PlatON 平台安全多方计算（MPC）的能力，实现了对计算数据的隐私保护。

**注意**：当前版本仅提供两方安全计算的隐私合约编写的规范，三方及多方隐私计算计算版本将陆续推出。

### 隐私计算对象

PlatON底层封装了对隐私计算对象，用来承接隐私数据，目前仅仅支持整型隐私计算对象，即`emp::Integer`，并提供了一些构造方法，将基础数据类型转化为隐私对象，然后对隐私对象进行运算操作实现隐私计算逻辑。 

- 构造方法

```cpp
Integer(const int8_t& value, int party = PUBLIC);
Integer(const int16_t& value, int party = PUBLIC);
Integer(const int32_t& value, int party = PUBLIC);
Integer(const int64_t& value, int party = PUBLIC);
	
//长整数构造，length表示位数，str表示十进制整数
Integer(int length, const string& str, int party = PUBLIC);
```

在提供的构造方法中，支持将8位、16位、32位、64位整型以及其他任意长度的整型转化为隐私计算对象，并指定对应的参与方，目前参与方只能为`emp::ALICE`或`emp::BOB`。 如声明32位有符号整型隐私计算对象方式如下：

```cpp
int32_t in=123;

emp::Integer v1(in, emp::ALICE); //声明Alice 隐私计算变量
emp::Integer v2(in, emp::BOB);   //声明Bob 隐私计算变量
```
- 操作符重载

```cpp
//位运算
Integer operator<<(int shamt)const;
Integer operator>>(int shamt)const;
Integer operator<<(const Integer& shamt)const;
Integer operator>>(const Integer& shamt)const;
	
//算术
Integer operator+(const Integer& rhs)const;
Integer operator-(const Integer& rhs)const;
Integer operator-()const;
Integer operator*(const Integer& rhs)const;
Integer operator/(const Integer& rhs)const;
Integer operator%(const Integer& rhs)const;
	
//逻辑运算
Integer operator&(const Integer& rhs)const;
Integer operator|(const Integer& rhs)const;
Integer operator^(const Integer& rhs) const;
```

对整型隐私对象`emp::Integer`提供了位运算、算术运算和逻辑运算的接口，使用方式如下：

```cpp
int32_t in=123;
emp::Integer v1(in, emp::ALICE); 
emp::Integer v2(in, emp::BOB);  
emp::Integer ret1 = v1 << 1;     
emp::Integer ret2 = v1 + v2;      
emp::Integer ret3 = (v1 & v2);
```

- 运算方法

```cpp
//比较emp::Integer大小
Bit geq(const Integer & rhs) const;
//判断两个emp::Integer是否相等
Bit equal(const Integer & rhs) const;
//将emp::Integer取绝对值
Integer abs() const;
```

对整型隐私对象`emp::Integer`提供了比较大小和绝对值运算接口，如比较两个整型隐私对象的差值做绝对值运算如下：

```cpp
int32_t in=123;
emp::Integer v1(in, emp::ALICE); 
emp::Integer v2(in, emp::BOB);  
emp::Integer ret1 = (v1 - v2).abs()
```

- 取值

```cpp
string reveal_string(int party = PUBLIC) const;
int64_t reveal(int party = PUBLIC) const;
uint64_t reveal_uint(int party = PUBLIC) const;
```

用于取出隐私对象中的值，转为相应的基础类型，使用方式如下：

```cpp
int32_t in=123;
emp::Integer v1(in, emp::ALICE); 
emp::Integer v2(in, emp::BOB); 
emp::Integer ret1 = (v1 - v2).abs()
int32_t ret = ret1.reveal()
```

- 其他

```cpp
int size() const;                        //计算隐私对象的位数
Bit& operator[](int index);              //取index上的值
const Bit & operator[](int index) const; //取index上的值
```

完整的隐私计算对象`emp::Integer`头文件可在[这里](https://github.com/kelvinskk/emp-tool/blob/8d86e41a8f5e234465698fe4da4a6730cf4e0cee/include/integer.h)查看。

**注意** :

- 隐私计算运算类型变量运算要求Alice和Bob在同类型、同长度的条件下进行，否则计算报错。
- 隐私计算运算类型变量运算要求计算双方，一个为Alice角色，另一个为Bob角色，否则计算无法进行。

### 隐私计算函数

计算逻辑的实现函数，输入参数为2个，参数1为Alice的输入，参数2为Bob的输入，暂时不支持更多参数。

函数定义原型：

```cpp
return_type function_name(declare_type in1, declare_type in2)
```

参数说明：

```
in1:           Alice方数据
in2:           Bob方的数据
declare_type:  入参类型
return_type:   返回类型
```

**注意：**

1. 入参类型和返回类型要求：

   - C++基础类型：char, unsigned, int, short, long, char, float, double等；
   - 字符串： std::string ；
   - 自定义类型：`protobuf`定义的消息类型；

2. 输入参数和返回值全部使用 `protobuf`编码，包括基础类型，string，自定义类型；

3. 在 [隐私数据服务开发包](#隐私数据服务开发包) 中，已经完成了基本数据类型的 `protobuf` 打包和解包处理。


### 使用范例

- 范例一：编写一个输入和返回为基础类型的隐私计算函数如下：

```cpp
 #include "integer.h"    

/**
* 比较Alice与Bob输入的大小
*/
bool YaoMillionairesProblem(int in1, int in2) {
    emp::Integer alice(in1, emp::ALICE);  	// Alice输入
    emp::Integer bob(in2, emp::BOB);       	// Bob输入
    int ret = (alice - bob).reveal();      	// 执行比较大小，并获取比较结果
    return ret >= 0;
}
```

- 范例二：编写一个带有`protobuf`自定义类型的隐私计算函数如下：

```cpp
#include "msg.pb.h"
#include "integer.h"   

/**
* 对Alice输入的Foo对象字段item1、item2分别和Bob的输入值in2进行绝对值运算，返回计算后的Foo对象
*/
Foo foo_abs(const Foo& in1, int32_t in2){
    Foo Foo = in1;
    emp::Integer item1(in1.item1(), emp::ALICE);    //Alice输入item1
    emp::Integer item2(in1.item2(), emp::ALICE);    //Alice输入item2
    emp::Integer v2(in2, emp::BOB);                 //Bob输入
    emp::Integer ret1 = (item1 - v2).abs();         //将Alice输入item1与Bob的输入做隐私计算后取绝对值
    emp::Integer ret2 = (item2 - v2).abs();         //将Alice输入item2与Bob的输入做隐私计算后取绝对值
    foo.set_item1(ret1.reveal());                   //重置item1
    foo.set_item2(ret2.reveal());                   //重置item2
    return foo;                                     
}
```

其中Foo定义如下：msg.proto

```protobuf
syntax = "proto3";
message Foo {
    int32 item1 = 1;
    int32 item2 = 2;
    string info = 3;
};
```

在隐私合约中使用自定义类型的引用之前，需要将`proto`文件编译生成`msg.pb.h`、`msg.pb.cc`、`Msg.java` 三个文件，其中`msg.pb.h`作为隐私合约编写的头文件，`msg.pb.cc`、`Msg.java` 在 `plang` 编译隐私合约时分别通过参数`-protobuf-cc`、`-protobuf-java`指定参与编译，且`Msg.java`在数据输入端和计算发起端时候需要引入。`proto`编译方式如下：

```bash
$ protoc msg.proto --cpp_out=./ --java_out=./ -I{path/to/protobuf/include}
```

其中 `{path/to/protobuf/include}`需替换为`protobuf`安装头文件路径。

## 隐私数据服务开发包

隐私数据服务开发包提供给隐私数据参与方使用，基于该SDK可以为MPC计算提供数据源，并最终获取计算上链，使用隐私数据服务开发包，只需要实现MPC回调接口类即可。

### MPC回调接口

```java
public interface MpcCallbackInterface {
    public byte[] input(final InputRequestPara para);                    // data input
    public void error(final InputRequestPara para, ErrorCode error);     // error notify
    public void result(final InputRequestPara para, final byte[] data);  // result notify
}
```
在用`plang`生成的客户端代码中会生成该接口的实现类，如下：

```java
abstract class MpcCallbackBase_fbbf2d8c40e87f406991b1d40bdd94dd implements MpcCallbackInterface {
        public abstract byte[] inputImpl(final InputRequestPara para);
        
        public byte[] input(final InputRequestPara para) {
            // TODO: do what you want to do, before call inputImpl
            return inputImpl(para);
        }

        public void error(final InputRequestPara para, ErrorCode error) {
            // TODO: do what you want to do

        }

        public void result(final InputRequestPara para, final byte[] data) {
            // TODO: do what you want to do
        }
 }
```

方法使用说明：

| 方法名 | 方法说明                                                     |
| ------ | ------------------------------------------------------------ |
| input  | 通过 inputImpl 方法传入数据参与计算的数据，可以直接通过 ret_value 给基础类型赋值，或者通过builder.set_XXX 给 message 结构体进行赋值 |
| error  | 计算过程出现的错误会通过该接口返回，用户可以在这里对错误信息进行处理 |
| result | 目前在计算参与Bob方能够通过该接口获得计算结果，其中`data`是明文的16进制串 |

更多示例参考[MPCSamples.java](privacy-contract/samples/MPCSamples.java)。

## 隐私客户端开发包

***说明：以下涉及的方法均在由 `plang` 编译获得代理类 `ProxyXXX.java` 中使用。***

### 发起计算

```java
public String startCalc(Method method)
public String startCalc(Method method, int retry)
```
函数功能：

* 通过该方法发起计算，并返回交易哈希，根据该交易哈希，可以获取计算任务ID、交易回执、结果密文等。

参数说明：

* `method`： 发起计算调用的方法，Method定义在生成的代理类中。
* `retry`： 重发次数，如果返回为null，会重发交易，直到返回的不为空，或者达到重发次数；默认retry为0。
* 返回值：交易哈希。

### 获取任务ID

```java
public String getTaskId(String transactionHash)
public String getTaskId(String transactionHash, long timeout)
```
函数功能：

* 根据交易哈希，获取计算任务ID，后续可以通过任务ID查询结果。

参数说明：

* `transactionHash`：发起计算方法`startCalc`返回的交易哈希。
* `timeout`：超时时间，最小0，最大180s。
* 返回值：任务ID。

### 获取交易回执

```java
public TransactionReceipt getTransactionReceipt(String transactionHash);
```

函数功能：

 * 根据交易哈希，获取交易回执。

参数说明：

* `transactionHash`：发起计算方法 `startCalc` 返回的交易哈希。
* 返回值：交易回执。如果计算成功，返回的交易回执中 `logs` 参数不为空，否则表示计算失败。`logs` 实例如下：

```json
[{
    address: "0xb136a7190dd23c57c1d8f07011174af3bbe3cfe2",
    blockHash: "0x7b59c274d6e4e67cbee9e92e754ada44e6ff88fe4de632680758a3d6eb9eecc0",
    blockNumber: 4977,
    data: "0xf84301b84064396361306232363866376163336339323732313565666234636335653038393339336631376631663336363765656666313664396536653836313866353137",
    logIndex: 0,
    removed: false,
    topics: ["0x9938524d0b7638d8cb06ef86cc46ae3074007aea703ce237efe1470b6bdb5f31"],
    transactionHash: "0x9bbf80f7d976e422472bd3cb3b96eb9fc71116c581da31da5102928db3fe4db3",
    transactionIndex: 0
}]
```

### 获取结果密文

```java
public String getResultByTransactionHash(String transactionHash)
public String getResultByTransactionHash(String transactionHash, long timeout)
public String getResultByTaskId(String taskId)
public String getResultByTaskId(String taskId, long timeout)
```

函数功能：

 * 根据交易hash或者任务ID，查询计算结果，结果是用计算发起方的公钥加密(ECIES)后的密文。

参数说明：

* `transactionHash`：发起计算方法 `startCalc` 返回的交易哈希。
* `taskId`： `getTaskId`返回的任务ID。
* `timeout`：超时时间，最小0，最大180s。
* 返回值：计算结果密文。

### 获取结果明文

```java
public int getInt32(String cipher)
public long getInt64(String cipher)
// getUInt32 getUInt64 getBool getFloat getDouble getString ... 
public com.abc.sample.Samples.Foo getFoo(String cipher)
// ...
```

函数功能：

* 该类接口将获取到的密文进行解密之后转换为相应的基本类型，或者自定义的 `protobuf` 结构体。其中基本类型的的转换在 `mpc-proxy-sdk` 中已经进行封装，而自定义的 `protobuf` 结构体的转换在 `plang` 编译获得的 JAVA 文件中生成，可以直接使用。

参数说明：

* `cipher`： 密文字符串。
* 返回值：基本类型或结构体。

**注意：由于该方法中需要将密文解析为明文，需要计算发起方才能通过该方法获取明文，非计算发起方调用此方法会因无法解密而报错！**

### 其他

```java
public static void showMethodMap()
```

函数功能：     

* 显示函数方法名，函数原型，函数枚举。

```java
public String getPlainText(String cipher)
```

函数功能：     

* 传入密文，获取结果明文(16进制字符串)。这个在知道私钥和密文的情况下即可使用。
