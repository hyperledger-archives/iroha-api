# Queries

## Query structure 

``` protobuf
 message Query {
    message Header {
        uint64 created_time = 1;
        Signature signature = 2;
    }
 
   Header header = 1;
   string creator_account_id = 2;

   oneof query {
        GetAccount get_account = 3;
        GetSignatories get_account_signatories = 4;
        GetAccountTransactions get_account_transactions = 5;
        GetAccountAssetTransactions get_account_asset_transactions = 6;
        GetAccountAssets get_account_assets = 7;
        GetRoles get_roles = 8;
        GetAssetInfo get_asset_info = 9;
        GetRolePermissions get_role_permissions = 10;
    }

    uint64 query_counter = 11;
  }
```

Each query consists of following things:
<ol>
    <li> Header </li>
    <li> Creator account_id </li>
    <li> Query (one of Query type) object </li>
    <li> Query counter </li>
</ol>

**Header** stores: 
<ul>
    <li>  time of creation (unix time, in milliseconds) </li> 
    <li>  single signature (ed25519 pubkey + signature — singed message with private key) </li> 
</ul>

**Creator account_id** stores account id in form username@domain <br>
**Query object** might be any of types, described below <br>
**Query counter** is used to prevent replay attack and it is formed on client side

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
    Permissions permissions = 3;
    uint32 quorum = 4;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id | username@domain
Domain name | domain of account id | ?
Permissions | set of allowed permissions for the account | ?
Quorum | number of signatories needed to sign the transaction to make it valid  | integer

<aside class="notice">It is better to rename `domain_name` into `domain_id` for consistency purposes</aside>

### Validation

?

## Get signatories

### Purpose

Purpose of _get signatories query is to get signatories, which act as an identity of the account.

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
Keys | an array of public keys | ed25519

### Validation

?

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
Transactions | an array of transactions for given account | ?

### Validation

?

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
Account ID | account id to request transactions from  | username@domain
Asset ID | asset id in order to filter transactions containing this asset | username@domain

#### Response

> Response 

```protobuf
message TransactionsResponse {
    repeated Transaction transactions = 1;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Transactions | an array of transactions for given account and asset | ?

### Validation

?

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
Account ID | account id to request balance from  | username@domain
Asset ID | asset id to know its balance | asset#domain

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
Asset ID | asset id to knows its balance | ?
Account ID | account which has this balance  | ?
Balance | balance of asset | ?

### Validation

?

## Get asset info

### Purpose

In order to know precision for given asset, and other related info in future, such as description, etc. user can send `GetAssetInfo` query to Iroha network 

Given this Query, in successful case Iroha returns `AssetResponse` with an infomation about the asset.

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
Asset ID | asset id to know related information | asset#domain

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
Asset ID | asset id to get information about | ?
Domain ID | domain related to this asset  | ?
Precision| number of digits after comma | ?

### Validation

?

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
Roles | array of created roles in the network | ?

### Validation

?

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
Role ID | role to get permissions for | ?

#### Response

> Response 

```protobuf
message RolePermissionsResponse {
    repeated string permissions = 1;
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Permissions | array of permissions related to the role | ?

### Validation

?