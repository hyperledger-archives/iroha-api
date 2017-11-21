# Run irohad locally

## Generate genesis block

1. Create peers.list file containing the peers' addresses. For the local node suffice to add only single peer:
```bash
$ echo 'localhost:10001' > peers.list
```

2. Generate genesis block using iroha-cli:
```bash
$ iroha-cli --genesis_block --peers_address peers.list
```

File genesis.block with keypair files for Administrator account (`admin@test.priv` and `admin@test.pub`) and node (`node0.priv` and `node0.pub`) should be generated in the same folder.

## Prepare config file

Configuration file keeps information about storage credentials and irohad parameters:

| Parameter         | Type    | Meaning                                                                                       |
|-------------------|---------|-----------------------------------------------------------------------------------------------|
| block_store_path  | string  | Path to store blocks of committed transactions                                                |
| torii_port        | integer | Port to access iroha node                                                                     |
| internal_port     | integer | Port used to communicate ordering service, YAC consensus and block loader for synchronization |
| pg_opt            | string  | Postgres credentials                                                                          |
| redis_host        | string  | Redis host's IP address                                                                       |
| redis_port        | integer | Port to access redis storage                                                                  |
| max_proposal_size | integer | Maximum size of created proposals                                                             |
| proposal_delay    | integer | The period of time (in ms) used to prepare proposal of transactions                           |
| vote_delay        | integer | The period of time (in ms) of spreading vote across the network                               |
| load_delay        | integer | The period of time (in ms) between synchronizations between peers                             |

Example:

```
{
  "block_store_path" : "/tmp/block_store/",
  "torii_port" : 50051,
  "internal_port" : 10001,
  "pg_opt" : "host=localhost port=5432 user=postgres password=mysecretpassword",
  "redis_host" : "localhost",
  "redis_port" : 6379,
  "max_proposal_size" : 10,
  "proposal_delay" : 5000,
  "vote_delay" : 5000,
  "load_delay" : 5000
}
```

## Launch irohad

To launch irohad daemon, following parameters must be passed:

| Parameter     | Meaning                                                                                      |
|---------------|----------------------------------------------------------------------------------------------|
| config        | configuration file, containing postgres, and redis connection, and values to tune the system |
| genesis_block | initial block in the ledger                                                                  |
| keypair_name  | private and public key file names without .priv or .pub extension. Used by peer to sign the blocks                             |

Use this command to launch iroha from development branch:

```
irohad --config example/config.sample --genesis_block example/genesis.block --keypair_name example/node
```
