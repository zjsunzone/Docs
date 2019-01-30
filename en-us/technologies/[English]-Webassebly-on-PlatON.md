## WebAssembly Introduction

WebAssembly (Wasm for short) is a new and efficient format for compiling to the web that is portable and efficient in terms of size and load time. This is a new platform-independent binary code format. Wasm can be compiled from high-level languages such as C++ and Rust.

## WebAssembly usage in PlatON

The main goal of Wasm is to solve JavaScript performance problems, but it can be even more useful for implementing blockchain smart contracts. Smart contracts can be written in high-level languages such as C++/Rust with vast arrays of existing libraries, thus significantly lowering the threshold for writing smart contracts.
No need to re-invent the wheel.

### Toolchain

PlatON has chosen C++ as  programming language for smart contracts and provides the following toolchain:

* clang : C/C++ front-end compiler responsible for generating LLVM intermediate bytecode, generated as `.bc` files
* llvm-link : The llvm linker is responsible for linking `.bc` files. Mainly links `.bc` files in the C/C++ standard library with the target `.bc` files.
* llc : Responsible for compiling `.bc` files into assembly files `.s`
* platon-s2wasm (s2wasm from binaryen): responsible for generating wast files from assembly code
* platon-wast2wasm (wasm-dis from binaryen): responsible for generating wasm files (binary format) from wast files (text format)

### Virtual machine
PlatON uses [life][1] as the PlatON virtual machine. [life][2] is the Go version of the Wasm virtual machine, inherited from [wagon][3]. Compared to wagon, life has a more comprehensive support for the standard specification. It is also more flexible at importing external functions. After modification by us, the [PlatON][4] virtual machine can use external functions on the chain and calculate [Gas][5].

  
### PlatON Wasm execution process

![Wasm_compile_pub_tx (1).jpg-68.8kB][6]

* Write a contract in C++
* Generate contract ABI files using the platon-abigen tool
* Generate `.bc` files with clang
* Connect all target `.bc` files with llvm-link
* Compile `.s` assembly files with llc
* Generate `.wast` files with platon-s2wasm
* Generate `.wasm` files using platon-wast2wasm
* Send the compiled ABI and Wasm files to PlatON using the modified Web3 client
* The contract's init function will be called during deployment to initialize the contract
* During the execution of a transaction startup parameters of the contract are initialized as described in the ABI files, then a virtual machine is created and the contract is executed

### PlatON Wasm optimization

The virtual machine normally parses the [Module][7] every time the contract is executed. This is very time-consuming due to wasm files usually being very large, and as the modules are always parsed the same way for the same contract, PlatON can cache the modules for each contract. There are two types of cache: in-memory and file. Caching is implemented in the following way:

* Parse the module each time a contract is deployed and then save the module to disk and cache it in memory at the same time
* The in-memory cache is designed as a priority queue
* Each time the virtual machine starts it loads the corresponding module according to the address of the contract.
* Modules are first fetched from the in-memory cache. In case of a cache miss, the module is obtained from disk and added to the in-memory cache.

  [1]: https://github.com/perlin-network/life
  [2]: https://github.com/perlin-network/life
  [3]: https://github.com/go-interpreter/wagon
  [4]: https://github.com/PlatONnetwork/PlatON-Go
  [5]: https://www.cryptocompare.com/coins/guides/what-is-the-gas-in-ethereum/
  [6]: http://static.zybuluo.com/tracebundy/r9699bipuourf5temlypw68x/wasm_compile_pub_tx%20%281%29.jpg
  [7]: https://webassembly.org/docs/modules/