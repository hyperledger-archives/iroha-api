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
       GetRolePermissions get_role_permissions = 10;
       GetAssetInfo get_asset_info = 11;
     }
     // used to prevent replay attacks.
     uint64 query_counter = 12;
  }

  Payload payload = 1;
  Signature signature = 2;
}
```

Each query consists of the following fields:
<ol>
    <li> Payload, which contains created time, id of account creator, query counter and a query object</li>
    <li> Signature, which signs payload </li>
</ol>

**Payload** stores everything except signatures:
<ul>
    <li> **Time of creation** (unix time, in milliseconds) </li>
    <li> **Creator account_id** stores account id in form username@domain  </li>
    <li> **Query counter** is used to prevent replay attack and it is formed on client side </li>
    <li> **Query object** might be of any type, described below </li>
</ul>

**Signatures** contain one or many signatures ([ed25519](https://ed25519.cr.yp.to) pubkey + signature).

## Validation

The validation for all commands includes:

 * timestamp — should not be older than 24 hours from the peer's time
 * signature of query creator — used for checking the identity of query creator
 * query counter — checked to be incremented with every subsequent query from query creator
 * roles — depending on the query creator's role, it can be applied to the same account, account in the domain, in the whole system or not allowed at all

## Get account

### Purpose

Purpose of _get account_ query is to get the state of an account.

This Query returns `AccountResponse`, which contains array with roles of the account and `Account` object with following fields:
<ul>
    <li>account_id </li>
    <li>domain_name </li>
    <li>permissions object </li>
    <li>quorum — the quantity of signatures to pass within transaction to execute commands for this account </li>
</ul>

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
            "pubkey": "407e57f50ca48969b08ba948171bb2435e035d82cec417e18e4a38f5fb113f83",
            "signature": "81744e004555970ad114a2b8f7a0d1bb087e26c6e009a6147781a5042dbbf8e00f1fd5a4d4ddb123c1c0813f00d633b7295e482a43001edbe7f51dd4d32aef05"
        },
    "created_ts": 1517560129182,
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
    repeated string account_roles = 2;
}
message Account {
    string account_id = 1;
    string domain_id = 2;
    uint32 quorum = 3;
    string json_data = 4;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id | username@domain
Domain ID | domain of account id | `[A-Za-z0-9]{1,9}`
Quorum | number of signatories needed to sign the transaction to make it valid  | integer
json_data | key-value information | properly formed json information

## Get signatories

### Purpose

Purpose of _get signatories_ query is to get signatories, which act as an identity of the account.

This Query returns `SignatoriesResponse`, which is one or many public [ed25519](https://ed25519.cr.yp.to) keys.

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
            "pubkey": "407e57f50ca48969b08ba948171bb2435e035d82cec417e18e4a38f5fb113f83",
            "signature": "81744e004555970ad114a2b8f7a0d1bb087e26c6e009a6147781a5042dbbf8e00f1fd5a4d4ddb123c1c0813f00d633b7295e482a43001edbe7f51dd4d32aef05"
        },
    "created_ts": 1517560129182,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetSignatories",
    "account_id": "test@test"
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id to request signatories  | username@domain

#### Response

> Response

```protobuf
message SignatoriesResponse {
    repeated bytes keys = 1;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Keys | an array of public keys | [ed25519](https://ed25519.cr.yp.to)

## Get account transactions

### Purpose

In a case when a list of transactions per account is needed, `GetAccountTransactions` query can be formed.

This Query returns `TransactionsResponse`, which is a collection of transactions.

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
            "pubkey": "407e57f50ca48969b08ba948171bb2435e035d82cec417e18e4a38f5fb113f83",
            "signature": "81744e004555970ad114a2b8f7a0d1bb087e26c6e009a6147781a5042dbbf8e00f1fd5a4d4ddb123c1c0813f00d633b7295e482a43001edbe7f51dd4d32aef05"
        },
    "created_ts": 1517560129182,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetAccountTransactions",
    "account_id": "test@test"
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id to request transactions from  | username@domain


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

`GetAccountAssetTransactions` query returns all transactions associated with given account and asset

This Query returns `TransactionsResponse`, which is a collection of transactions.

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
            "pubkey": "407e57f50ca48969b08ba948171bb2435e035d82cec417e18e4a38f5fb113f83",
            "signature": "81744e004555970ad114a2b8f7a0d1bb087e26c6e009a6147781a5042dbbf8e00f1fd5a4d4ddb123c1c0813f00d633b7295e482a43001edbe7f51dd4d32aef05"
        },
    "created_ts": 1517560129182,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetAccountAssetTransactions",
    "account_id": "test@test",
    "asset_id": "coin#test"
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id to request transactions from  | username@domain
Asset ID | asset id in order to filter transactions containing this asset | asset_name#domain

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

`GetTransactions` is used for retrieving information about transactions, based on their hashes.

This Query returns `TransactionsResponse`, which is a collection of transactions.

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
            "pubkey": "407e57f50ca48969b08ba948171bb2435e035d82cec417e18e4a38f5fb113f83",
            "signature": "81744e004555970ad114a2b8f7a0d1bb087e26c6e009a6147781a5042dbbf8e00f1fd5a4d4ddb123c1c0813f00d633b7295e482a43001edbe7f51dd4d32aef05"
        },
    "created_ts": 1517560129182,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetTransactions",
    "tx_hashes": [bf6edb882d53f5532cb416455834878db7af08fb814f8c95d6867a6d9eea4057]
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

This Query returns `AccountAssetResponse`.

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
            "pubkey": "407e57f50ca48969b08ba948171bb2435e035d82cec417e18e4a38f5fb113f83",
            "signature": "81744e004555970ad114a2b8f7a0d1bb087e26c6e009a6147781a5042dbbf8e00f1fd5a4d4ddb123c1c0813f00d633b7295e482a43001edbe7f51dd4d32aef05"
        },
    "created_ts": 1517560129182,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetAccountAssets",
    "account_id": "test@test",
    "asset_id": "coin#test"
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id to request balance from  | username@domain
Asset ID | asset id to know its balance | asset_name#domain

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
Asset ID | identifier of asset used for checking the balance | asset_name#domain
Account ID | account which has this balance  | username@domain
Balance | balance of asset | > 0

## Get asset info

### Purpose

In order to know precision for given asset, and other related info in the future, such as a description of the asset, etc. user can send `GetAssetInfo` query.

This Query returns `AssetResponse` with an information about the asset.

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
            "pubkey": "407e57f50ca48969b08ba948171bb2435e035d82cec417e18e4a38f5fb113f83",
            "signature": "81744e004555970ad114a2b8f7a0d1bb087e26c6e009a6147781a5042dbbf8e00f1fd5a4d4ddb123c1c0813f00d633b7295e482a43001edbe7f51dd4d32aef05"
        },
    "created_ts": 1517560129182,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetAssetInfo",
    "asset_id": "coin#test"
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Asset ID | asset id to know related information | asset_name#domain

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
Asset ID | asset id to get information about | `[A-Za-z0-9]{1,9}`
Domain ID | domain related to this asset  | `[A-Za-z0-9]{1,9}`
Precision| number of digits after comma | uint8_t

## Get roles

### Purpose

To get existing roles in the system, a user can send `GetRoles` query to Iroha network.

This Query returns `RolesResponse` containing an array of existing roles.

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
            "pubkey": "407e57f50ca48969b08ba948171bb2435e035d82cec417e18e4a38f5fb113f83",
            "signature": "81744e004555970ad114a2b8f7a0d1bb087e26c6e009a6147781a5042dbbf8e00f1fd5a4d4ddb123c1c0813f00d633b7295e482a43001edbe7f51dd4d32aef05"
        },
    "created_ts": 1517560129182,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetRoles"
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

## Get role permissions

### Purpose

To get available permissions per role in the system, a user can send `GetRolePermissions` query to Iroha network.

This Query returns `RolePermissionsResponse` containing an array of permissions related to the role.

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
            "pubkey": "407e57f50ca48969b08ba948171bb2435e035d82cec417e18e4a38f5fb113f83",
            "signature": "81744e004555970ad114a2b8f7a0d1bb087e26c6e009a6147781a5042dbbf8e00f1fd5a4d4ddb123c1c0813f00d633b7295e482a43001edbe7f51dd4d32aef05"
        },
    "created_ts": 1517560129182,
    "creator_account_id": "admin@test",
    "query_counter": 1,
    "query_type" : "GetRolePermissions",
    "role_id" : "MoneyCreator"
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
