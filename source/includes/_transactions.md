# Transactions

> Transaction 

```protobuf
message Transaction {
	Header header = 1;
	Meta meta = 2;
	Body body = 3;
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

Each transaction consists of three things:
<ul>
    <li> Header </li> 
    <li> Meta information </li> 
    <li> Body </li> 
</ul>

> Header

```protobuf
message Header {
	uint64 created_time = 1;
	repeated Signature signatures = 2;
}
```
```json 
{
	"created_ts": int(13),
    "signatures": [
        {
            "pubkey": string(64),
            "signature": string(128),
        }
    ], …
}
```

> Meta 

>_creator_account_id_ is used to prevent replay attacks: 

```protobuf
message Meta {
	string creator_account_id = 1;
	uint64 tx_counter = 2;
}
```
```json
{
	"creator_account_id": string(?),
    "tx_counter": int, …
}
```

> Body 

```protobuf
message Body {
	repeated Command commands = 1;
}
```
```json
{
	"commands": [
        {
            "command_type": string(?),
			/* other command-specific fields */
		}
	]
}
```
**Header** stores: 
<ul>
    <li> Time of creation (unix time, in milliseconds) </li> 
    <li> One or many signatures (ed25519 pubkey + signature - base64 from Transaction model) </li> 
</ul>

**Meta** information stores:
<ul>
    <li> ID of transaction creator (username@domain) </li>
    <li> Transaction counter — counts how many times transaction creator send transactions in total. Counter is used to prevent replays, and is formed on client side. </li>
</ul>

<aside class="notice">
In preview version, there is no way to get current transaction counter by any means. You have to remember it on a client side and increment it. In future version this will be changed, and client application may retrieve this information.
</aside>

**Body** stores repeated commands, which would be described in details later.