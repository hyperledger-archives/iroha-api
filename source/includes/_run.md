# Run Irohad on a local machine

## Generate genesis block

1. Create peers.list file containing the peers' addresses. For the local node suffice to add only single peer:
```bash
$ echo 'localhost:10001' > peers.list
```
2. Generate genesis block using iroha-cli:
```bash
$ iroha-cli --genesis_block --peers_address peers.list
```
3. File `genesis.block` should be generated in the same folder.

Pass generated genesis block to irohad using --genesis_block flag.

## Launch irohad

To launch iroha the following flags are used:

| Flag                | Meaning                                      | Type   | Default        |
|---------------------|----------------------------------------------|--------|----------------|
| --create            | Create new ledger with given genesis block   | bool   | false          |
| --dbpath            | Define path to folder for block storage      | string | ""             |
| --genesis_block     | Define path to genesis block                 | string | "genesis.json" |
| --postgres_host     | Define host on which postgres is listening   | string | "localhost"    |
| --postgres_port     | Define port on which postgresql is listening | int32  | 5432           |
| --postgres_username | Define postgres user                         | string | "postgres"     |
| --private_key       | Define path to private key                   | string | ""             |
| --public_key        | Define path to public key                    | string | ""             |
| --redis_host        | Define host on which redis is listening      | string | "localhost"    |
| --redis_port        | Define port on which redis is listening      | int32  | 6379           |
| --torii_host        | Define host for iroha to listen on           | string | "0.0.0.0"      |
| --torii_port        | Define port for iroha to listen on           | int32  | 50051          |


### Launch irohad using command line arguments
Usage:
```bash
$ irohad -flag1=val -flag2=val
```

### Launch irohad using environment variables

Usage:
```bash
$ export FLAGS_flag1=val
$ export FLAGS_flag2=val
$ irohad --fromenv=flag1,flag2
```

### Launch irohad using FLAG FILE
Usage:
```bash
$ cat /tmp/flags
--flag1=val
--flag2=val
$ irohad --flagfile=/tmp/flags
```
