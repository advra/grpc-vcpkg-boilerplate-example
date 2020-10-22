# Boilerplate Hello World gRPC Example with Vcpkg Manager

## 0. Purpose
GRPC is a high performance open source rpc framework. By default gRPC must be installed natively. Over the years this method has slowly deprecated with other methods of installation. While they recommend other methods such as `Bazel` or `vcpkg`, their examples are not exactly plug and play. This repository is an attempt to try and provide a template that build and compiles with `vcpkg` manager as a starting point which you can build upon from.

## 1. Pre-Environment Setup
This project requires `vcpkg` a package manager to be installed. VCPKG is used to manage your packages dependencies for projects like Protobufs and gRPC. If you already have vcpkg manager installed then continue to [2. Compile](https://github.com/advra/grpc-vcpkg-boilerplate-example/blob/main/README.md#2-compile), otherwise continuing reading.

### 1.1 Installing vcpkg
Run the following commands to clone and install vcpkg in the current directory.
```
sudo apt-get install -y unzip build-essential
git clone https://github.com/microsoft/vcpkg
cd vcpkg/
./bootstrap-vcpkg.sh -disableMetrics
```
### 1.2 Install packages
In the vcpkg folder, search and install for `grpc` and `protobufs`. You may find that installing grpc will also install protobuf by default. 
```
./vcpkg search grpc
./vcpkg install grpc && ./vcpkg install protobuf
```
Once installed you can run `./vcpkg list` to ensure all the packages have successfully installed.

### 1.3 Vcpkg user-wide install
Next we will expose the installed packages to our project. From vcpkg's documentation "the recommended and most productive way to use vcpkg is via user-wide integration, making the system available for all projects you build. The user-wide integration will prompt for administrator access the first time it is used on a given machine, but afterwards is no longer required and the integration is configured on a per-user basis." ['(Source: Vcpkg Documentation)'](https://vcpkg.readthedocs.io/en/latest/examples/installing-and-using-packages/).
```
./vcpkg integrate install
```
Running command will output the path which we need to supply to cmake each time we build with a project. Take note of the `DCMAKE_TOOLCHAIN_FILE` flag as we will use this later in `2. Compile`.
```
aa@ubuntu:~/vcpkg$ ./vcpkg integrate install
Applied user-wide integration for this vcpkg root.

CMake projects should use: "-DCMAKE_TOOLCHAIN_FILE=/home/aa/vcpkg/scripts/buildsystems/vcpkg.cmake"
```

### 1.4 Configure CMake
OPTIONAL: Skip this if you already have system cmake installed. 

By default ubuntu / raspbian and other OS may come with an older version of CMake so you may need to upgrade. Vcpkg downloads Cmake for you but you will need to add it to your `$PATH` as shown below. If you do not have CMake installed you can download the appropriate binaries from [Cmake's download page](https://cmake.org/download/). If running this setup from an OS other than Linux, Windows or OSX then you will need to either download the binaries or compile from source with yoru appropriate OS [(Read More)](https://cmake.org/install/).

Note: Replace XX to your version of CMake! The path below may change depending on the file binary you downloaded/compiled.
```
export PATH=$PATH:$HOME/vcpkg/downloads/tools/cmake-X.X.X-linux/cmake-X.X.X-Linux-x86_64/bin
```

## 2. Compile
Update `$VCPKG_HOME` defined in the CMakelist.txt (located at the root) to match the location you installed VCPKG to.
```
set (VCPKG_HOME "$ENV{HOME}/vcpkg")
```
Once set we can then build the project and pass the `DCMAKE_TOOLCHAIN_FILE` direction pointing to our VCPKG's cmake previously installed. 
```
mkdir build && cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=/home/aa/vcpkg/scripts/buildsystems/vcpkg.cmake
make
```
If you get an error when running cmake.. such as below you may need to target your appropriate OS with `-DVCPKG_TARGET_TRIPLET`.
```
  Could not find a package configuration file provided by "gRPC" with any of
  the following names:

    gRPCConfig.cmake
    grpc-config.cmake
```
Here are some targets for your OS:

windows: `cmake -DVCPKG_TARGET_TRIPLET=x86-windows ..`

linux: `cmake -DVCPKG_TARGET_TRIPLET=x64-linux ..`

raspberrypi: `cmake -DVCPKG_TARGET_TRIPLET=arm-linux ..`

### 3. Run
You can find our compiled binaries in `build/src`. Start up the server with `./greeter_server` to begin listening for connections. Then open another terminal to and run  `./greeter_client` and/or `./greeter_client2`. Similarly, you can run the asynchronous examples with `./greeter_async_server` and `./greeter_async_client` in seperate terminals. 

