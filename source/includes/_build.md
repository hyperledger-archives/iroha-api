# Build

To launch Iroha peer network, an executable daemon should be launched. Currently we support Unix-like systems (we are basically targeting at popular Linux distros and macOS), while Windows remains to be a challenge for community and we don't have any plans to support it so far.

If you happen to have Windows or you don't want to spend time figuring out troubles with dependencies on your system — please use Docker environment, made with care for new contributors of Iroha (and hostages of Windows).

## Docker

Please, use latest available docker daemon and docker-compose. If you have troubles launching Docker container — it is likely that you have an outdated version.

### Clone Git repository

Clone Iroha [repository](https://github.com/hyperledger/iroha) in any convenient folder on your machine.

> Cloning Iroha

``` shell
git clone -b develop --depth=1 \
https://github.com/hyperledger/iroha
```

### How to run development environment

Run the script `run-iroha-dev.sh`, contained in the folder `scripts`: `sh .../iroha/scripts/run-iroha-dev.sh`

After you execute this script, following things happen:

 1. The script checks if you don't have containers with Iroha already running. It ends up with reattaching you to interactive shell upon succesful completion.
 2. The script will download iroha-docker-develop, redis and postgres images. Iroha image contains all development dependencies, and is based on top of ubuntu:16.04.
 3. Three containers are created and launched.
 4. The user is attached to the interactive environment for development and testing with `iroha` folder mounted from the host machine. Iroha folder is mounted to `/opt/iroha` in Docker container.

<aside class="notice">
Docker environment will be removed when you detach from the container.
</aside>


### Build project and run tests

> Build

``` bash
cmake -H. -Bbuild;
cmake --build build -- -j$(nproc)
```

<aside class="notice">
On macOS $(nproc) variable does not work. Check number of logical cores with <code>sysctl -n hw.ncpu</code>
</aside>

Run shell commands from "Build" section on the right. Built binaries (`irohad` and `iroha-cli`) will be in `./build/bin` directory.

After you built the project — please run tests to check the operability of the daemon.

### Cmake parameters

We use CMake to build platform-dependent build files. It has numerous of the flags for configuring the final build. Note that beside the listed params [cmake's vars](https://cmake.org/Wiki/CMake_Useful_Variables) can be useful as well. Also as long as this page can be deprecated (or just not complete) you can browse custom flags via `cmake -L`, `cmake-gui`, or `ccmake`.

<aside class="notice">
You can specify parameters at the <code>cmake</code> configuring stage (e.g <code>cmake -DTESTING=ON</code>).
</aside>

#### Main params

Parameter | Possible values | Default | Description
----------|-----------------|------------|--------------------
TESTING | ON/OFF | ON | Enables or disables build of the tests
BENCHMARKING | ON/OFF | OFF |  Enables or disables build of the benchmarks
COVERAGE | ON/OFF | OFF | Enables or disables coverage
SWIG_PYTHON | ON/OFF | OFF | Enables or disables libraries and native interface bindings for python
SWIG_JAVA | ON/OFF | OFF | Enables or disables libraries and native interface bindings for java

#### Packaging specific params

Parameter | Possible values | Default | Description
----------|-----------------|------------|--------------------
ENABLE_LIBS_PACKAGING | ON/OFF | ON | Enables or disables all types of packaging
PACKAGE_ZIP | ON/OFF | OFF | Enables or disables zip packaging
PACKAGE_TGZ | ON/OFF | OFF | Enables or disables tar.gz packaging
PACKAGE_RPM | ON/OFF | OFF | Enables or disables rpm packaging
PACKAGE_DEB | ON/OFF | OFF | Enables or disables deb packaging

### Add to irohad and iroha-cli to path (optional)

`export PATH=.../iroha/build/bin:$PATH`

<aside class="notice">
Write this to your shell configuration to make it persistent.
</aside>

> How to run tests method #1

``` shell
cmake --build build --target test
```

> How to run tests method #2

``` shell
ctest . # in build folder
```

### Execute `iroha-cli` with `irohad` running

Execute `run-iroha-dev.sh` again to attach to existing container.

## Linux or macOS

> Launching Docker and Postgres in Docker

``` shell
docker run --name some-redis 
    -p 6379:6379\
    -d redis:3.2.8

docker run --name some-postgres 
    -e POSTGRES_USER=postgres \
    -e POSTGRES_PASSWORD=mysecretpassword \
    -p 5432:5432 \
    -d postgres:9.5
```

To launch Iroha daemon, running postgres and redis services are required. You may launch them on your local machine, or use docker containers, as provided on the right side.

### Linux (debian-based)

To install dependencies, clone, and build the project, please use this code:
<script src="https://gist.github.com/neewy/39557aa444c3317aeffbdedd9f0f51e2.js"></script>

#### Outdated dependencies 

In case that some dependencies are outdated in apt, you are advised to use following sources to install dependencies:

##### Boost
To install Boost libraries (`libboost-all-dev`), use [current release](http://www.boost.org/users/download/) from Boost webpage, or use [debian repository](https://packages.debian.org/sid/libboost-all-dev) for latest library.
The only dependencies are system and filesystem, so use `./bootstrap.sh --with-libraries=system,filesystem` when you are building the project.

##### CMake
To install CMake tool (`cmake`), use [latest release](https://cmake.org/download/) from CMake webpage.

### macOS

> macOS brew

``` shell
brew tap soramitsu/iroha
brew install iroha
```

To install dependencies, clone, and build the project, please use this code:
<script src="https://gist.github.com/neewy/bc0fea777592a5381aa4ab6e68bfe699.js"></script>
