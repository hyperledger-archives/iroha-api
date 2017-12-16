# Run the daemon (irohad) 

## Create genesis block

A genesis block is the first block in the chain.

<b>Important:</b> each peer in the initialized network should have exactly the same genesis block, so create it once and distribute across peers. This block may have <b>any</b> data, as no-one validates it in the peer network.

### Using iroha-cli

#### Create list of peers
Create peers.list file containing network addresses of peers' internal ports in the network (a single peer address goes on new line. If you want to run Iroha on single peer, it is sufficient to mention only localhost: <br> `$ echo 'localhost:10001' > peers.list`

Composing `peers.list` for multiple peers is a bit trickier at the moment: the first line should contain the address of a peer, which is the ordering service (and the address should include internal port). The remaining addresses are the network addresses of all other peers in the network, including the torii (external API) port. 

As an example, for 4 peers the file will look like the following:
<ul>
  <li>10.128.13.1:10001</li>
  <li>10.128.13.2:50051</li>
  <li>10.128.13.3:50051</li>
  <li>10.128.13.4:50051</li>
</ul>

where `10.128.13.1:10001` is the address of the first peer, acting as an ordering service, while the rest of the list are peers' addresses with Torii port.

<aside class="notice">
Internal port is used for communication of internal components. Each peer potentially may have different, adjustable internal port numbers.
</aside>

#### Use iroha-cli
Generate genesis block using iroha-cli (see shell command on the right side).

Iroha-cli tool is used to generate keypairs for every peer, mentioned in the peers list. This is done only for convenience — in production environment please consider using manual or more trusted approach.

> Generating genesis block

```bash
$ iroha-cli --genesis_block \ 
--peers_address peers.list
```
Executing this command will result in generation of following files:

* `admin@test.priv` and `admin@test.pub` — keypair for user with admin and asset creator rights
* `test@test.priv` and `test@test.pub` — keypair for user with user rights
* `node0.priv` and `node0.pub` — keypair for first node, mentioned in the `peers.list`
* If mentioned more than one peer address in `peers.list`: `nodeX.priv` and `nodeX.pub` — keypair for #X node, mentioned in the peers.list, (where X is starts from 1) 
* `genesis.block` — genesis block itself

<aside class="notice">
What are those asset creator, admin, user rights? Iroha supports role-based access control, so initial roles for accounts should be created in genesis block. Check <a href="#permission-model">permission model</a> section of the documentation to find out more. 
</aside>

### Manually

Follow JSON structure of the block, as it is reflected in [JSON schema for the block](https://gist.github.com/neewy/4e347bc8493951733998e01af633dbbb).

## Prepare configuration file

Configuration file keeps information about storage credentials and irohad parameters:

| Parameter         | Type    | Meaning                                                                                       |
|-------------------|---------|-----------------------------------------------------------------------------------------------|
| block_store_path  | string  | Path to store blocks of committed transactions (flat file storage)                            |
| torii_port        | integer | Port to access iroha node gRPC (default 50051)                                                |
| internal_port     | integer | Port for communication between ordering service, YAC consensus and block loader for synchronization (default 10001) |
| pg_opt            | string  | Postgres credentials                                                                          |
| redis_host        | string  | Redis host IP address                                                                       |
| redis_port        | integer | Port to access redis storage                                                                  |
| max_proposal_size | integer | Maximum size of created proposals                                                             |
| proposal_delay    | integer | The period of time (in ms) used to prepare proposal of transactions                           |
| vote_delay        | integer | The period of time (in ms) of spreading vote across the network                               |
| load_delay        | integer | The period of time (in ms) between synchronizations between peers                             |

Example:

<script src="https://gist.github.com/neewy/1ebe653de074dac1675f74eb19824b2e.js"></script>

## Launch irohad

To launch irohad daemon, following parameters must be passed:

| Parameter     | Meaning                                                                                      |
|---------------|----------------------------------------------------------------------------------------------|
| config        | configuration file, containing postgres, and redis connection, and values to tune the system |
| genesis_block | initial block in the ledger                                                                  |
| keypair_name  | private and public key file names without file extension. Used by peer to sign the blocks |

Use this command to launch iroha:

`irohad --config example/config.sample --genesis_block example/genesis.block --keypair_name example/node0`
