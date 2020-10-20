# Boilerplate GRPC's Hello World With VCPKG

### 0. Purpose
grpc is a high performance open source rpc framework. They recommend users to install gRPC natively. Over the years this method has slowly been deprecated. While they recommend other methods such as vcpkg, their examples are not exactly plug and play. This is an attempt to try and provide examples which work with `vcpkg` manager and provide boilerplate example which you can build off of.

### 1. Pre-Environment Setup
This project requires `vcpkg a package manager` to already be installed. VCPKG is used to manage your packages dependencies for projects like Protobufs and gRPC. If you already have vcpkg manager installed then continue to `2. How to Use` otherwise continuing reading on.

#### 1.1 Installing vcpkg
Running the following commands to clone and install vcpkg in the current directory.
```
sudo apt-get install -y unzip build-essential
git clone https://github.com/microsoft/vcpkg
cd vcpkg/
./bootstrap-vcpkg.sh -disableMetrics
```
#### 1.2 Install packages
When in the directory, search and install for `grpc` and `protobufs`. You may find that installing grpc will also install protobuf by default. 
```
./vcpkg search grpc
./vcpkg install grpc && ./vcpkg install protobuf
```
Once install you may run `./vcpkg list` to list all the packages that have successfully installed.

#### 1.3 Vcpkg user-wide install
Next we will expose the installed packages to our project. From vcpkg's documentation "the recommended and most productive way to use vcpkg is via user-wide integration, making the system available for all projects you build. The user-wide integration will prompt for administrator access the first time it is used on a given machine, but afterwards is no longer required and the integration is configured on a per-user basis." ['- vcpkg documentation'](https://vcpkg.readthedocs.io/en/latest/examples/installing-and-using-packages/).
```
./vcpkg integrate install
```
Running command will output the path which we need to supply to cmake each time we build with a project. Take note of the `DCMAKE_TOOLCHAIN_FILE` flag as we will use this later in `2. Copmile`.
```
aa@ubuntu:~/vcpkg$ ./vcpkg integrate install
Applied user-wide integration for this vcpkg root.

CMake projects should use: "-DCMAKE_TOOLCHAIN_FILE=/home/aa/vcpkg/scripts/buildsystems/vcpkg.cmake"
```

#### 1.4 Configure CMake
Skip this if you already have system cmake installed. By default ubuntu comes with an older version so you may need to install it. vcpkg downloads cmake for you, but you will need to modify and export the `PATH` variable (as shown below). If you do not have CMake installed you can download the appropriate binaries from ['cmake's download page'](https://cmake.org/download/). If running this setup from an OS other than Linux, Windows or OSX then you will need to compile from source for your OS ['read more'](https://cmake.org/install/).
Note: Ensure to export the correct version of cmake, this will only export version 3.17.2
```
export PATH=$PATH:$HOME/vcpkg/downloads/tools/cmake-3.17.2-linux/cmake-3.17.2-Linux-x86_64/bin
```

### 2. Compile
This package assumes you have `vcpkg` located at your `$HOME/vcpkg`. If it is located somewhere else in your system then set `VCPKG_HOME` to match the same install path for line 3 in the CMakeLists.txt file as shown:
```
set (VCPKG_HOME "$ENV{HOME}/vcpkg")
```
Once the `VCPKG_HOME` variable is set, we can then build the project and pas the `DCMAKE_TOOLCHAIN_FILE` direction pointing to our VCPKG's cmake previously installed.
```
mkdir build && cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=/home/aa/vcpkg/scripts/buildsystems/vcpkg.cmake
make
```

### 3. Run
And thats it! You can then run the examples by going to the `build/src` directory and run the binaries. In one terminal start up the server with `./greeter_server` and open another terminal to execute `./greeter_client` and `./greeter_client2`. Similarly, you can run the asynchronous examples running `./greeter_async_server` in one terminal and `./greeter_async_client` in another. 

