# Transactions

> Transaction 

```protobuf
message Transaction {
  Payload payload = 1;
  repeated Signature signature = 2;
 }
```
```json
{
	/* Transaction */
	"signatures": array of objects,
    "created_ts": int(13),
    "creator_account_id": string(?),
    "tx_counter": int,
    "commands": array of objects
}
```


To provide connectivity with Iroha, currently Google RPC is used. To form a transaction with one or many commands, or send a query JSON format is used in CLI.

Client application should follow descibed protocol and form transactions accordingly to description below.

## Transaction structure 

Each transaction consists of two parts:
<ul>
    <li> Payload </li> 
    <li> Signature </li> 
</ul>

> Payload

```protobuf
message Payload {
  repeated Command commands = 1;
  string creator_account_id = 2;
  uint64 tx_counter  = 3;
  uint64 created_time = 4;
}
```
```json
{
    "commands": [
        {
            "command_type": string(?),
            /* other command-specific fields */
        }
    ],
    "creator_account_id": string(?),
    "tx_counter": int,
    "created_ts": int(13)
}
```

> Signature 

> one or more signatures of payload

```protobuf
message Signature {
   bytes pubkey    = 1;
   bytes signature = 2;
}
```
```json 
{
    "signatures": [
        {
            "pubkey": string(64),
            "signature": string(128),
        }
    ], …
}
```

**Payload** stores everything except signatures: 
<ul>
    <li> Time of creation (unix time, in milliseconds) </li> 
    <li> ID of transaction creator (username@domain) </li>
    <li> Transaction counter — counts how many times transaction creator send transactions in total. Counter is used to prevent replays, and is formed on client side. </li>
    <li> Repeated commands which would be described in details later</li> 
</ul>

**Signatures** contain one or many signatures (ed25519 pubkey + signature):

<aside class="notice">
In preview version, there is no way to get current transaction counter by any means. You have to remember it on a client side and increment it. In future version this will be changed, and client application may retrieve this information.
</aside>
