## Installation on Windows 
The Windows environment supports three installation modes:

- [Chocolatey](#Chocolatey-based-Installation)
- [Binary package](#Binary-package-based-Installation)
- [Source code](#Source-code-based-Installation)

> Note：When installing through official binary packages and Chocolate, the native CPU architectures need to be above 2.0 and 2.0, otherwise PlatON needs to be installed by compiling and encoding.

### Chocolatey based Installation
We use the Chocolatey package manager to install the required build tools. If you don’t have Chocolatey, follow the instructions on [https://chocolatey.org](https://chocolatey.org).

Start PowerShell as an administrator and install PlatON using the choco command:
```
> choco install platonnetwork --version=0.5.0
```
After complation, you will find `platon,ethkey` in the default installation path `C:\ProgramData\chocolatey\bin`.

### Binary package based Installation
Windows version of the Platon binary can been download from [here](https://download.platon.network/0.5/platon-windows-x86_64-0.5.0.zip). No installation is required after downloading, and it can be used directly by decompression.

The extracted files should be as following:

- `platon` client executable file
- `ethkey key` generator
- `ctool` wasm contract deploy kit

### Source code based Installation

The Windows build environment requires:

- git:`2.19.1`
- go language development kit: `go(1.11+)`
- mingw: `mingw(V6.0.0)`
- cmake: `3.0+`

> Note: You can install the above compilation environment by yourself. Before compiling the `Platon` source code, make sure that the above environment works properly. The `chocolatey` installation can also be used, referring to the following ways.

#### 1. Install compilation environment
Start PowerShell as an administrator and install Platon using the choco command:

```
// install git
> choco install git
// install golang
> choco install golang
// install mingw
> choco install mingw
// install cmake
> choco install cmake
```
Most of the software installed with the Chocolatey package manager use the default installation path, although some publishers override the default settings in their software. Installing these packages will modify the PATH environment variable. The final installation path can be found in PATH.

#### 2. Get `PlatON` source code
Create `src/github.com/PlatONnetwork/` and `bin` directories under the current `%GOPATH%` directory and clone the source code of `PlatON-GO` under the `PlatONnetwork` directory:
```
> git clone https://github.com/PlatONnetwork/PlatON-Go.git --recursive
```

#### 3. Compile

> Note: The following commands need to be run in the `Git-bash` environment.

Enter the `PlatON-GO` directory:
```
> cd PlatON-GO
```

Before compiling the project, we need to run the following scripts in the PlatON-GO directory to compile the libraries:
```
> ./build/build_deps.sh
```

Run the following scripts in the source directory `PlatON-GO` to generate the `platon`、`ethkey` and `ctool`:
```
> go run build/ci.go install ./cmd/platon
> go run build/ci.go install ./cmd/ethkey
> go run build/ci.go install ./cmd/ctool
```

After compilation, the `platon`、 `ethkey` and `ctool` executable files will be generated in the `PlatON-Go/build/bin` directory，and then copy these executable files to your own working directory.

> Note: Repeated compilation will overwrite the previously generated executable.











