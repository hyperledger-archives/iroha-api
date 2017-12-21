# Queries

## Query structure

``` protobuf
message Query {
  message Payload {
    uint64 created_time = 1;
    string creator_account_id = 2;
     oneof query {
       GetAccount get_account = 3;
       GetSignatories get_account_signatories = 4;
       GetAccountTransactions get_account_transactions = 5;
       GetAccountAssetTransactions get_account_asset_transactions = 6;
       GetTransactions get_transactions = 7;
       GetAccountAssets get_account_assets = 8;
       GetRoles get_roles = 9;
       GetAssetInfo get_asset_info = 10;
       GetRolePermissions get_role_permissions = 11;
     }
     // used to prevent replay attacks.
     uint64 query_counter = 12;
  }

  Payload payload = 1;
  Signature signature = 2;
}
```

Each query consists of following things:
<ol>
    <li> Payload, which contains one created time, id of account creator, query counter and a query of one of possible types </li>
    <li> Signature, which signs payload </li>
</ol>

**Payload** stores everything except signatures:
<ul>
    <li> **Time of creation** (unix time, in milliseconds) </li>
    <li> **Creator account_id** stores account id in form username@domain  </li>
    <li> **Query counter** is used to prevent replay attack and it is formed on client side </li>
    <li> **Query object** might be any of types, described below </li>
</ul>

**Signatures** contain one or many signatures (ed25519 pubkey + signature):

## Get account

### Purpose

Purpose of _get account_ query is to get state of an account.

Given this Query, in successful case Iroha returns `AccountResponse`, which contains Account object with following fields:
<ul>
    <li>account_id </li>
    <li>domain_name </li>
    <li>permissions object </li>
    <li>quorum — the quantity of signatures to pass within transaction to execute commands for this account </li>
</ul>

<aside class="warning"> `AccountResponse` should not contain array of permissions, but an array of `Role` object instead. It will be reworked. </aside>

### Structure


#### Query

> Query

``` protobuf
message GetAccount {
    string account_id = 1;
}
```
```json
{
    "signature":
        {
            "pubkey": "…",
            "signature": "…"
        },
    "created_ts": …,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetAccount",
    "account_id": "test@test"
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id to request its state  | username@domain


#### Response

> Response

```protobuf
message AccountResponse {
    Account account = 1;
}
message Account {
    string account_id = 1;
    string domain_name = 2;
    uint32 quorum = 3;
    string json_data = 4;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id | `[a-z]{1,9}\@[a-z]{1,9}`
Domain name | domain of account id | `[a-z]{1,9}`
Quorum | number of signatories needed to sign the transaction to make it valid  | integer
json_data | key-value information | properly formed json information

<aside class="notice">It is better to rename `domain_name` into `domain_id` for consistency purposes</aside>

### Validation

Stateless validation (check timestamp and signature)

## Get signatories

### Purpose

Purpose of _get signatories_ query is to get signatories, which act as an identity of the account.

Given this Query, in successful case Iroha returns `SignatoriesResponse`, which is one or many public ed25519 keys.

### Structure


#### Query

> Query

``` protobuf
message GetSignatories {
    string account_id = 1;
}
```
```json
{
    "signature":
        {
            "pubkey": "…",
            "signature": "…"
        },
    "created_ts": …,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetSignatories",
    "account_id": "test@test"
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id to request signatories  | `[a-z]{1,9}\@[a-z]{1,9}`


#### Response

> Response

```protobuf
message SignatoriesResponse {
    repeated bytes keys = 1;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Keys | an array of public keys | ed25519

### Validation

Stateless validation (check timestamp and signature)

## Get account transactions

### Purpose

In case when a list of transactions per user is needed, `GetAccountTransactions` query can be formed.

Given this Query, in successful case Iroha returns `TransactionsResponse`, which is a collection of transactions.

<aside class="notice">Team of Iroha maintainers is implementing pagination for this query, so that user can retrive transactions partially, using tx_hash as head, and quantity as size of page</aside>

### Structure

#### Query

> Query

``` protobuf
message GetAccountTransactions {
    string account_id = 1;
}
```
```json
{
    "signature":
        {
            "pubkey": "…",
            "signature": "…"
        },
    "created_ts": …,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetAccountTransactions",
    "account_id": "test@test"
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id to request transactions from  | `[a-z]{1,9}\@[a-z]{1,9}`


#### Response

> Response

```protobuf
message TransactionsResponse {
    repeated Transaction transactions = 1;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Transactions | an array of transactions for given account | Committed transaction

## Get account asset transactions

### Purpose

When an information about transaction per user using certain asset is needed, `GetAccountAssetTransactions` query is sent.

Given this Query, in successful case Iroha returns `TransactionsResponse`, which is a collection of transactions.

### Structure

#### Query

> Query

``` protobuf
message GetAccountAssetTransactions {
    string account_id = 1;
    string asset_id = 2;
}
```
```json
{
    "signature":
        {
            "pubkey": "…",
            "signature": "…"
        },
    "created_ts": …,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetAccountAssetTransactions",
    "account_id": "test@test",
    "asset_id": "coin#test"
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id to request transactions from  | `[a-z]{1,9}\@[a-z]{1,9}`
Asset ID | asset id in order to filter transactions containing this asset | `[a-z]{1,9}\#[a-z]{1,9}`

#### Response

> Response

```protobuf
message TransactionsResponse {
    repeated Transaction transactions = 1;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Transactions | an array of transactions for given account and asset | Committed transactions

## Get transactions

### Purpose

`GetTransactions` serves for retrieving information about transactions by its hashes.

Given this Query, in successful case Iroha returns `TransactionsResponse`, which is a collection of transactions.

### Structure

#### Query

> Query

```protobuf
message GetTransactions {
    repeated bytes tx_hashes = 1;
}
```
```json
{
    "signature":
        {
            "pubkey": "…",
            "signature": "…"
        },
    "created_ts": …,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetTransactions",
    "tx_hashes": [string(64),…]
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Transactions hashes | transactions' hashes to request transactions from  | (proto) array of 32 bytes, (json) 64 size of hex strings

#### Response

> Response

```protobuf
message TransactionsResponse {
    repeated Transaction transactions = 1;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Transactions | an array of transactions for given account and asset | Committed transactions

## Get account assets

### Purpose

To know the state of an asset per account (balance), `GetAccountAssets` query can be formed.

Given this Query, in successful case Iroha returns `AccountAssetResponse`.

### Structure

#### Query

> Query

``` protobuf
message GetAccountAssets {
 string account_id = 1;
 string asset_id = 2;
}
```

```json
{
    "signature":
        {
            "pubkey": "…",
            "signature": "…"
        },
    "created_ts": …,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetAccountAssets",
    "account_id": "test@test",
    "asset_id": "coin#test",
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id to request balance from  | `[a-z]{1,9}\@[a-z]{1,9}`
Asset ID | asset id to know its balance | `[a-z]{1,9}\#[a-z]{1,9}`

#### Response

> Response

```protobuf
message AccountAsset {
    string asset_id = 1;
    string account_id = 2;
    Amount balance = 3;
}

message AccountAssetResponse {
    AccountAsset account_asset = 1;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Asset ID | asset id to knows its balance | `[a-z]{1,9}\#[a-z]{1,9}`
Account ID | account which has this balance  | `[a-z]{1,9}\@[a-z]{1,9}`
Balance | balance of asset | > 0

### Validation

Stateless validation (check timestamp and signature)

## Get asset info

### Purpose

In order to know precision for given asset, and other related info in future, such as description, etc. user can send `GetAssetInfo` query to Iroha network

Given this Query, in successful case Iroha returns `AssetResponse` with an information about the asset.

### Structure

#### Query

> Query

``` protobuf
message GetAssetInfo {
    string asset_id = 1;
}
```
```json
{
    "signature":
        {
            "pubkey": "…",
            "signature": "…"
        },
    "created_ts": …,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetAssetInfo",
    "asset_id": "coin#test",
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Asset ID | asset id to know related information | `[a-z]{1,9}\#[a-z]{1,9}`

#### Response

> Response

```protobuf
message AssetResponse {
Asset asset = 1;
}

message Asset {
    string asset_id = 1;
    string domain_id = 2;
    uint32 precision = 3;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Asset ID | asset id to get information about | `[a-z]{1,9}`
Domain ID | domain related to this asset  | `[a-z]{1,9}`
Precision| number of digits after comma | uint8_t

### Validation

Stateless validation (check timestamp and signature)

## Get roles

### Purpose

To get available roles in the system, user can send `GetRoles` query to Iroha network.

Given this Query, in successful case Iroha returns `RolesResponse` containing array of existing roles.

### Structure

#### Query

> Query

``` protobuf
message GetRoles {
}
```
```json
{
    "signature":
        {
            "pubkey": "…",
            "signature": "…"
        },
    "created_ts": …,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetRoles",
}
```

#### Response

> Response

```protobuf
message RolesResponse {
    repeated string roles = 1;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Roles | array of created roles in the network | vector of strings describing existing roles in the system

### Validation

Stateless validation (check timestamp and signature)

## Get role permissions

### Purpose

To get available permissions per role in the system, user can send `GetRolePermissions` query to Iroha network.

Given this Query, in successful case Iroha returns `RolePermissionsResponse` containing array of permissions related to the role.

### Structure

#### Query

> Query

``` protobuf
message GetRolePermissions {
    string role_id = 1;
}
```
```json
{
    "signature":
        {
            "pubkey": "…",
            "signature": "…"
        },
    "created_ts": …,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetRolePermissions",
    "role_id" : "MoneyCreator",
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Role ID | role to get permissions for | string describing role existing in the system

#### Response

> Response

```protobuf
message RolePermissionsResponse {
    repeated string permissions = 1;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Permissions | array of permissions related to the role | string of permissions related to the role

### Validation

Stateless validation (check timestamp and signature)
