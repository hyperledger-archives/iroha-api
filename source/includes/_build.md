# Build

## Docker 

### Git repository

Clone `iroha` repository on your directory.

> Cloning Iroha

``` bash
git clone -b develop --depth=1 \
https://github.com/hyperledger/iroha /path/to/iroha
```

### How to run development environment

``` bash
/path/to/iroha/scripts/run-iroha-dev.sh
```

You will be attached to interactive environment for development and testing with `iroha` folder mounted from host machine.

Docker environment will be removed when you logout from the container.

### Build `iroha` and run tests

Build:
``` bash
cmake -H. -Bbuild; 
cmake --build build -- -j$(nproc)
```

`irohad` and `iroha-cli` binaries will be in `./build/bin` directory.

> How to run tests method #1

``` bash
cmake --build build --target test
```

> How to run tests method #2

``` bash
ctest . # in build folder 
```

### Execute `iroha-cli` with `irohad` running

Execute `run-iroha-dev.sh` again to attach to existing container.

## Linux or macOS

> Linux

``` bash
apt-get -y --no-install-recommends install \
        build-essential python-software-properties \
        automake libtool \
        libssl-dev zlib1g-dev libboost-all-dev \
        libc6-dbg golang \
        git ssh tar gzip ca-certificates \
        python3 python3-pip python3-setuptools \
        lcov \
        wget curl cmake file unzip gdb \
        iputils-ping vim ccache \
        gcovr vera++ cppcheck doxygen \
        graphviz graphviz-dev; \
    apt-get -y clean
```    

> macOS

``` bash
brew …
```