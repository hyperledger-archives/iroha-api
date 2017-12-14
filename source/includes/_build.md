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
https://github.com/hyperledger/iroha /path/to/iroha
```

### How to run development environment

Run the script `run-iroha-dev.sh`, contained in the folder `scripts`: `sh .../iroha/scripts/run-iroha-dev.sh`

After you execute this script, following things happen:

 1. It checks if you don't have containers with Iroha already running. If yes — it will reattach you to interactive shell. Assuming if you don't have any, then:
 2. The script will download iroha-docker-develop, redis and postgres images. Iroha image contains all development dependencies and is based on top of ubuntu:16.04.
 3. Three containers are created and launched.
 4. User is attached to interactive environment for development and testing with `iroha` folder mounted from host machine.

<aside class="notice">
Docker environment will be removed when you logout from the container.
</aside>


### Build project and run tests

> Build

``` bash
cmake -H. -Bbuild;
cmake --build build -- -j$(nproc)
```

<aside class="notice">
On macOS $(nproc) variable does not work. Check number of logical cores with sysctl -n hw.ncpu
</aside>

We use CMake to build platform-dependent build files. Run shell commands from "Build" section on the right. Built binaries (`irohad` and `iroha-cli`) will be in `./build/bin` directory.

After you built whole project — please run tests to check the operability of the daemon.

### Add to irohad and iroha-cli to path (optional)

export PATH=/path/to/iroha/build/bin:$PATH

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

To launch Iroha daemon, running postgres and redis services are required. You may launch them on your local machine, or use docker containers, as provided in the right side.

> Launching Docker and Postgres in Docker

``` shell
docker run --name some-redis \
-p 6379:6379 -d redis:3.2.8
docker run --name some-postgres \
-e POSTGRES_USER=iroha \
-e POSTGRES_PASSWORD=helloworld \
-p 5432:5432 -d postgres:9.5
```

### Linux (debian-based)

To install dependencies, clone and build the project please [use this gist](https://gist.github.com/neewy/39557aa444c3317aeffbdedd9f0f51e2).


### macOS

> macOS brew

``` shell
brew tap soramitsu/iroha
brew install iroha
```

To install dependencies, clone and build the project please [use this gist](https://gist.github.com/neewy/bc0fea777592a5381aa4ab6e68bfe699).