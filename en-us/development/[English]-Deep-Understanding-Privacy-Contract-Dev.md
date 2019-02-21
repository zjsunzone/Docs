[TOC]

## Overview

This document mainly provides detailed instructions for the use of privacy contract, privacy data SDK, and privacy client SDK to help users understand the development of privacy contract.

## Privacy Contract

A **Private Contract** is a special MPC contract on [PlatON](#). It is developed using the `C++` language and runs on the MPC Virtual Machine (MPC VM) on PlatON nodes. Private contracts provide secure multi-party computing (MPC) services by leveraging the power of the PlatON platform's secure multi-party computing (MPC) to keep data secure and private.

**Note**: The current version only provides specifications for writing private contracts for two parties. Versions that supports private computation with three and more parties will be released soon.

### Privacy Computation Object

`PlatON` encapsulates the private computing object, which is used to undertake private data. Currently, it only supports the integer privacy computing object, namely `emp::Integer`, and provides some constructor to convert the basic data type into a privacy object and then perform arithmetic operations to implement privacy calculation logic.


- Construction method

```cpp
Integer(const int8_t& value, int party = PUBLIC);
Integer(const int16_t& value, int party = PUBLIC);
Integer(const int32_t& value, int party = PUBLIC);
Integer(const int64_t& value, int party = PUBLIC);
	
// Long integer construct, length represents the number of bits, and str represents a decimal integer
Integer(int length, const string& str, int party = PUBLIC);
```

In the provided constructor, it supports the conversion of 8-bit, 16-bit, 32-bit, 64-bit integer and other arbitrary length integers into privacy computing objects, and specifies the corresponding participants. Currently, the participants can only be `emp::ALICE` or `emp::BOB`. The way to declare a 32-bit signed integer privacy calculation object is as follows:

```cpp
int32_t in=123;

Emp::Integer v1(in, emp::ALICE); //Declare Alice privacy calculation variables
Emp::Integer v2(in, emp::BOB); //Declare Bob privacy calculation variables
```
- Operator overloading

```cpp
//bit operation
Integer operator<<(int shamt)const;
Integer operator>>(int shamt)const;
Integer operator<<(const Integer& shamt)const;
Integer operator>>(const Integer& shamt)const;
	
//arithmetic
Integer operator+(const Integer& rhs)const;
Integer operator-(const Integer& rhs)const;
Integer operator-()const;
Integer operator*(const Integer& rhs)const;
Integer operator/(const Integer& rhs)const;
Integer operator%(const Integer& rhs)const;
	
//logic operation
Integer operator&(const Integer& rhs)const;
Integer operator|(const Integer& rhs)const;
Integer operator^(const Integer& rhs) const;
```

An interface for bit operations, arithmetic operations, and logic operations is provided for the integer privacy object `emp::Integer`, which is used as follows:

```cpp
int32_t in=123;
emp::Integer v1(in, emp::ALICE); 
emp::Integer v2(in, emp::BOB);  
emp::Integer ret1 = v1 << 1;     
emp::Integer ret2 = v1 + v2;      
emp::Integer ret3 = (v1 & v2);
```

- Algorithm

```cpp
//Compare emp::Integer size
Bit geq(const Integer & rhs) const;
//Determine whether two emp::Integer are equal
Bit equal(const Integer & rhs) const;
//Take the emp::Integer absolute value
Integer abs() const;
```

An interface for comparison size and absolute value operation is provided for the integer privacy object `emp::Integer`, such as comparing the difference between two integer privacy objects to do the absolute value operation as follows:


```cpp
int32_t in=123;
emp::Integer v1(in, emp::ALICE); 
emp::Integer v2(in, emp::BOB);  
emp::Integer ret1 = (v1 - v2).abs()
```

- Get value

```cpp
string reveal_string(int party = PUBLIC) const;
int64_t reveal(int party = PUBLIC) const;
uint64_t reveal_uint(int party = PUBLIC) const;
```

Used to retrieve the value in the privacy object and convert it to the corresponding base type, using the following:

```cpp
int32_t in=123;
emp::Integer v1(in, emp::ALICE); 
emp::Integer v2(in, emp::BOB); 
emp::Integer ret1 = (v1 - v2).abs()
int32_t ret = ret1.reveal()
```

- Other

```cpp
int size() const;                        //calculate the number of bits in the privacy object
Bit& operator[](int index);              //take the value on index
const Bit & operator[](int index) const; //take the value on index
```

The complete privacy calculation object `emp::Integer` header file can be viewed [here](https://github.com/kelvinskk/emp-tool/blob/8d86e41a8f5e234465698fe4da4a6730cf4e0cee/include/integer.h).


**Note**:

* The variables ALICE and BOB in privacy computation must of the same type and length, otherwise an error will be thrown.
* In order to perform a computation with privacy variables, all corresponding computing parties must partake in the computation; in this case Alice and Bob.


### Privacy Computation Methods

Methods used in privacy computation take two parameters. Parameter one corresponds to the input of Alice, and parameter two corresponds to the input of Bob. Currently, additional parameters are not supported.

***Function definition prototype：***

```cpp
return_type function_name(declare_type in1, declare_type in2)
```

Parameter description:

```
in1:           Alice's input
in2:           Bob's input
declare_type:  Input type
return_type:   Return type
```

**Note**:

1. The allowed input parameter and return types in privacy computation methods are:
   - Basic types: char, unsigned, int, short, long, char, float, double, etc.
   - Strings: std::string
   - User-defined types: [protobuf](#Dependent component) defined message type
2. Writing classes for privacy computation:
   - Return values are all written using the [protobuf](#Dependent component), including basic types, strings and user-defined types
   - Input parameters are all written using [protobuf](#Dependent components), including basic types, strings, and user-defined types  
3. [MPC Java SDK] automatically handles the packing and unpacking of `protobuf` basic data types.


### Examples

- Example 1: Write an input and return as a basic type of privacy calculation function as follows:

```cpp
 #include "integer.h"    

/**
* Compare the size of Alice and Bob input
*/
bool YaoMillionairesProblem(int in1, int in2) {
    emp::Integer alice(in1, emp::ALICE);  	// Alice input
    emp::Integer bob(in2, emp::BOB);       	// Bob input
    int ret = (alice - bob).reveal();      	// Execute the comparison size and get the comparison result
    return ret >= 0;
}
```

- Example 2: Write a privacy calculation function with a custom type of `protobuf` as follows:

```cpp
#include "msg.pb.h"
#include "integer.h"   

/**
* Perform absolute value calculation on the Foo object fields item1 and item2 input by Alice and the input value in2 of Bob, and return the calculated Foo object
*/
Foo foo_abs(const Foo& in1, int32_t in2){
    Foo Foo = in1;
    emp::Integer item1(in1.item1(), emp::ALICE);    //Alice inputs item1
    emp::Integer item2(in1.item2(), emp::ALICE);    //Alice inputs item2
    emp::Integer v2(in2, emp::BOB);                 //Bob input
    emp::Integer ret1 = (item1 - v2).abs();         //Take Alice's item1 and Bob for privacy calculation and take the absolute value
    emp::Integer ret2 = (item2 - v2).abs();         //Take Alice's item2 and Bob for privacy calculation and take the absolute value
    foo.set_item1(ret1.reveal());                   //Reset item1
    foo.set_item2(ret2.reveal());                   //Reset item2
    return foo;                                     
}
```

Foo is defined as follows: msg.proto

```protobuf
syntax = "proto3";
message Foo {
    int32 item1 = 1;
    int32 item2 = 2;
    string info = 3;
};
```

Before using a custom type reference in a privacy contract, you need to compile the `proto` file to generate three files: `msg.pb.h`, `msg.pb.cc`, and `Msg.java`, where `msg.pb.h` is the header file written as a privacy contract, `msg.pb.cc` and `Msg.java` which are specified to participate in the compilation by the parameters `-protobuf-cc`, `-protobuf-java` when the plang compiles the privacy contract, and `Msg.java` needs to be introduced at the data input end and the calculation initiator. `.proto` is compiled as follows:


```bash
$ protoc msg.proto --cpp_out=./ --java_out=./ -I{path/to/protobuf/include}
```

Where `{path/to/protobuf/include}` needs to be replaced with the protobuf installation header path.

## Privacy Data Service SDK
The privacy Data Service SDK is provided to participants of privacy computations. The SDK can be used to provide data sources for MPC computations and to retrieve computation results. To use the Privacy Data Service SDK you only need to implement the MPC callback interface class.

### MPC callback interface

```java
public interface MpcCallbackInterface {
    public byte[] input(final InputRequestPara para);                    // data input
    public void error(final InputRequestPara para, ErrorCode error);     // error notify
    public void result(final InputRequestPara para, final byte[] data);  // result notify
}
```

The implementation class for this interface is generated in the client code generated by `plang`, as follows:

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

Method instructions:

| Method | Description                                                                                                                                                                                 |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| input  | Passing data for computation using the inputImpl method, you can assign the value to the basic type directly via ret_value, or assign the value to a message structure via builder.set_XXX. |
| error  | Errors in the computation process are returned via this interface, where the user can process the error message.                                                                            |
| result | The computation participant Bob can obtain the computation result through this interface, where `data` is a hexadecimal string in cleartext.                                                |


More examples reference [MPCSamples.java](privacy-contract/samples/MPCSamples.java)。


## privacy Client SDK

**Note: The methods involved below are all used in the proxy class `ProxyXXX.java` compiled by `plang`.**

### Initiating computations

```java
public String startCalc(Method method)
public String startCalc(Method method, int retry)
```

Function:

* This method is used to initiate a computation and returns a transaction hash. Using the transaction hash you can get the computation task ID, transaction receipt, the encrypted results etc. 

Description:

* `method`: Method called to initiate the computation. Method is defined in the generated proxy class.
* `retry`: Number of retries. If the return value is null, the transaction will be sent again until the return is not empty, or until the number of retries is reached. The default value of retry is 0.
* Returns a transaction hash

### Getting the task ID

```java
public String getTaskId(String transactionHash)
public String getTaskId(String transactionHash, long timeout)
```

Function:

* Uses a transaction ID to retrieve its computation ID. This can later be used to retrieve the computation results.

Description:

* `transactionHash`: The transaction hash as returned by the method `startCalc`
* `timeout`: Timeout, minimum 0, maximum 180s
* Returns the task ID

### Getting a transaction receipt

```java
public TransactionReceipt getTransactionReceipt(String transactionHash);
```

Function:

* Get a transaction receipt using the transaction hash.

Description:

* `transactionHash`: The transaction hash as returned by the method `startCalc`
* Returns the transaction receipt

**Note: If the computation was successful, the log key in the returned transaction receipt will not be empty. See example below:**


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

### Getting results in encrypt text

```java
public String getResultByTransactionHash(String transactionHash)
public String getResultByTransactionHash(String transactionHash, long timeout)
public String getResultByTaskId(String taskId)
public String getResultByTaskId(String taskId, long timeout)
```

Function:

* Query the computation result using the transaction hash or task ID. The result is encrypted (by ECIES) using the public key belonging to the party that initiated the computation.

Description:

* `transactionHash`: Transaction hash as returned by the `startCalc` method which was used to initiate the computation
* `taskId`: The task ID returned by `getTaskId`
* `timeout`: Timeout, minimum 0, maximum 180s
* Returns the encrypted result


### Getting results in cleartext

```java
public int getInt32(String cipher)
public long getInt64(String cipher)
// getUInt32 getUInt64 getBool getFloat getDouble getString ... 
public com.abc.sample.Samples.Foo getFoo(String cipher)
// ...
```

Function:

* This interface decrypts the obtained encrypted text and converts it to the corresponding basic type, or a custom `protobuf` structure. The code for converting basic types is encapsulated in `mpc-proxy-sdk`, and the code for converting the custom `protobuf` structures is generated in the Java file obtained by the `plang` compilation, and can be used directly.
  

Description:

* `cipher`: Encrypted string
* Returns a basic type or structure

**Note: Only the party who initiated the computation can call this method, which decrypts the encrypted text into cleartext. If anyone else calls this method it will throw an error!**

### Other

```java
public static void showMethodMap()
```
Description:

* Displays method names, method definitions, method enumerations.


```java
public String getPlainText(String cipher)
```

Description:

* Takes an encrypted string as parameter and returns a hexadecimal cleartext string. This can be used when you have the private key and the encrypted result.

