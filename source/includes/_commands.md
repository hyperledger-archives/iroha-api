# Commands

A command changes the state, called World State View, by performing an action over an entity (asset, account) in the system. Any command should be included in a transaction to perform an action.

## Add asset quantity

### Purpose

The purpose of _add asset quantity_ is to increase the quantity of an asset on account of transaction creator. Use case scenario is to increase the number of mutable asset in the system, which can act as a claim on a commodity (e.g. money, gold, etc.)

### Structure

```protobuf
message AddAssetQuantity {
    string account_id = 1;
    string asset_id = 2;
    Amount amount = 3;
}

message uint256 {
   uint64 first = 1;
   uint64 second = 2;
   uint64 third = 3;
   uint64 fourth = 4;
}

message Amount {
   uint256 value = 1;
   uint32 precision = 2;
}

```
```json
{
    "commands": [
        {
            "command_type": "AddAssetQuantity",
            "account_id": "test@test",
            "asset_id": "coin#test",
            "amount": {
                "value": string,
                "precision": int
            }
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id in which to add asset | account_name@domain
Asset ID  | id of the asset | asset#account
Amount  | positive amount of the asset to add | > 0

### Validation

1. Asset and account should exist
2. Added quantity precision should be equal to asset precision
3. Creator of transaction should have role which has permissions for issuing assets
4. Creator of transaction adds account quantity to his/her account only

## Add peer

### Purpose

The purpose of _add peer_ is to write into ledger the fact of peer addition into the peer network. After the peer was added, consensus and synchronization components will start to use it.

### Structure

```protobuf
message AddPeer {
    string address = 1;
    bytes peer_key = 2;
}
```
```json
{
    "commands": [
        {
            "command_type": "AddPeer",
            "address": "192.168.1.1:50001",
            "peer_key": string(64)
        },
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Address | resolvable address in network (IPv4, IPv6, domain name, etc.) | should be resolvable
Peer key | peer public key, which will be used in consensus algorithm to sign-off vote, commit, reject messages | ed25519 public key


### Validation

1. Creator of the transaction has a role which has CanAddPeer permission.
2. Such network address has not been already added.


## Add signatory

### Purpose

The purpose of _add signatory_ is to add an identifier to the account. Such identifier is a public key of another device or a public key of another user.

### Structure

```protobuf
message AddSignatory {
    string account_id = 1;
    bytes public_key = 2;
}
```
```json
{
    "commands": [
        {
            "command_type": "AddSignatory",
            "account_id": "test@test",
            "public_key": string(64)
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | Account to which to add signatory | account_name@domain
Public key | Signatory to add to account | ed25519 public key

### Validation

Two cases:
Case 1. Transaction creator wants to add signatory to own account, having permission CanAddSignatory
Case 2. Transaction creator was granted with CanAddSignatory permission to add signatory to this account

<aside class="notice">
Granting specific rights in the system allows other account to perform actions over the account, which granted such rights. They can be revoked if needed, and for more information — please check <a href="#permission-model">permission model</a> section.
</aside>

## Append role

### Purpose

The purpose of _append role_ is to promote an account to some created role in the system, where role is a set or permissions account has to perform an action (command or query).

### Structure

```protobuf
message AppendRole {
   string account_id = 1;
   string role_name = 2;
}
```
```json
{
    "commands": [
        {
            "command_type": "AppendRole",
            "account_id": "takemiya@test",
            "role_name": "Administrator"
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | id or account to append role to | already existent, account_name@domain
Role name | name of already created role | already existent, `[A-Za-z0-9_]{1,7}`

### Validation

1. Role should exist in the system
2. Transaction creator should have permissions to append role (CanAppendRole)
3. Account, which appends role, has set of permissions in his roles bigger or equal to the size of permission set of appended role.

## Create account

### Purpose

The purpose of _create account_ is to make entity in the system, capable of sending transactions or queries, storing signatories, personal data and identifiers.

### Structure

```protobuf
message CreateAccount {
    string account_name = 1;
    string domain_id = 2;
    bytes main_pubkey = 3;
}
```
```json
{
    "commands": [
        {
            "command_type": "CreateAccount",
            "account_name": "makoto.takemiya",
            "domain_id": "test",
            "main_pubkey": string
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account name | domain-unique name for account | A string in domain-name syntax defined in RFC1035. An account name is a list of labels separated by a period `.`. A label is a sequence of characters in `[a-zA-Z-]`. The length of a label must not exceed 63 characters
Domain ID | target domain to make relation with | should be created before the account, `[0-9A-Za-z]{1,9}`
Main pubkey| first public key to add to the account | ed25519 public key

### Validation

1. Transaction creator has permission to create account
2. Domain, passed as domain_id, has already been created in the system
3. Such public key has not been added before as first public key of account or added to multi-signature account 

## Create asset

### Purpose

The purpose of _сreate asset_ is to create a new type of asset, unique in a domain. An asset is a countable representation of a commodity. 

### Structure

```protobuf
message CreateAsset {
    string asset_name = 1;
    string domain_id = 2;
    uint32 precision = 3;
}
```
```json
{
    "commands": [
        {
            "command_type": "CreateAsset",
            "asset_name": "usd",
            "domain_id": "test",
            "precision": "2"
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Asset name | domain-unique name for asset | `[A-Za-z0-9]{1,9}`
Domain ID | target domain to make relation with | should be created before the asset, `[A-Za-z0-9]{1,9}`
Precision | number of digits after comma/dot | 0 <= precision <= uint32 max

### Validation

1. Transaction creator has permission to create assets
2. Asset name is unique per domain

## Create domain

### Purpose

The purpose of _create domain_ is to make new domain in Iroha network, which is a group of accounts.

### Structure

```protobuf
message CreateDomain {
    string domain_id = 1;
    string default_role = 2;
}
```
```json
{
    "commands": [
        {
            "command_type": "CreateDomain",
            "domain_id": "test2",
            "default_role": "User"
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Domain ID | ID for created domain | unique, `[0-9A-Za-z]{1,9}`
Default role | role for any created user in the domain | one of the existing roles

### Validation

1. _domain id_ is unique
2. Account, who sends this command in transaction, has role with permission to create domain
3. Role, which will be assigned to created user by default

## Create role

### Purpose

The purpose of _create role_ is to create new role in the system from the set of permissions.
Combining different permissions into roles, maintainers of Iroha peer network can create customized security model.

### Structure

```protobuf
message CreateRole {
   string role_name = 1;
   repeated string permissions = 2;
}
```
```json
{
    "commands": [
        {
            "command_type": "CreateRole",
            "role_name": "MoneyCreator",
            "permissions": [
                "CanAddAssetQuantity",
                …
            ]
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Role name | name of role to create | `[A-Za-z0-9_]{1,7}`
Permissions | array of already existent permissions | set of passed permissions is fully included into set of existing permissions  

### Validation

1. Set of passed permissions is fully included into set of existing permissions, and is not empty

## Detach role

### Purpose

The purpose of _detach role_ is to detach a role from the set of roles of an account.
By executing this command it is possible to decrease the number of possible actions in the system for the user.

### Structure

```protobuf
message DetachRole {
    string account_id = 1;
    string role_name = 2;
}
```
```json
{
    "commands": [
        {
            "command_type": "DetachRole",
            "account_id": "test@test",
            "role_name": "user"
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | ID of account where role will be deleted | already existent, account_name@domain
Role name | detached role | existing role

### Validation

1. Role is existing in the system
2. Account has such role

## Grant permission

### Purpose

The purpose of _grant permission_ is to give another account rights to perform actions over account of transaction sender (give someone right to do something with my account).

### Structure

```protobuf
message GrantPermission {
   string account_id = 1;
   string permission_name = 2;
}
```
```json
{
    "commands": [
        {
            "command_type": "GrantPermission",
            "account_id": "takemiya@soramitsu",
            "permission_name": "CanAddAssetQuantity"
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | id of account whom rights are granted | already existent, account_name@domain
Permission name | name of granted permission | permission is defined

### Validation

1. Account exists
2. Transaction creator is permitted to grant this permission

## Remove signatory

### Purpose

Purpose of _remove signatory_ is to remove public key, associated with an identity, from an account

### Structure

```protobuf
message RemoveSignatory {
    string account_id = 1;
    bytes public_key = 2;
}
```
```json
{
    "commands": [
        {
            "command_type": "RemoveSignatory",
            "account_id": "takemiya@test",
            "public_key": string(64)
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | ID of account to delete signatory from | already existent, account_name@domain
Public key | Signatory to delete | ed25519 public key

### Validation

When signatory is deleted, we should check if invariant of **size(signatories) >= quorum** holds.
Signatory should be added previously to the account.

Two cases:
Case 1. When transaction creator wants to remove signatory from their account and he or she has permission CanRemoveSignatory
Case 2. Transaction creator was granted with CanRemoveSignatory permission to remove signatory from this account

## Revoke permission

### Purpose

The purpose of _revoke permission_ is to revoke or dismiss given grant permission to another account in the network.

### Structure

```protobuf
message RevokePermission {
   string account_id = 1;
   string permission_name = 2;
}
```
```json
{
    "commands": [
        {
            "command_type": "RevokePermission",
            "account_id": "takemiya@soramitsu",
            "permission_name": "CanAddAssetQuantity"
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | id of account whom rights were granted | already existent, account_name@domain
Permission name | name of revoked permission | permission is defined

### Validation

1. Transaction creator should have previously granted this permission to target account

## Set account detail

### Purpose

Purpose of _set account detail_ is to set key-value information for a given account

### Structure

```protobuf
message SetAccountDetail{
    string account_id = 1;
    string key = 2;
    string value = 3;
}
```
```json
{
    "commands": [
        {
            "command_type": "SetAccountDetail",
            "account_id": "takemiya@soramitsu",
            "key": "position",
            "value": "Co-CEO"
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | id of account whom key-value information was set | already existent, account_name@domain
Key | key of information being set | `[A-Za-z0-9_]{1,}`
Value | value of corresponding key | -

### Validation

Two cases:
Case 1. When transaction creator wants to set account detail to his/her account and he or she has permission CanSetAccountInfo
Case 2. Transaction creator was granted with CanSetAccountInfo permission to set details about this account

## Set account quorum

### Purpose

The purpose of _set account quorum_ is to set the number of signatories required to confirm the identity of user, who creates the transaction. Use case scenario is to set the number of different users, utilizing single account, to sign off the transaction.

### Structure

```protobuf
message SetAccountQuorum {
    string account_id = 1;
    uint32 quorum = 2;
}
```
```json
{
    "commands": [
        {
            "command_type": "SetAccountQuorum",
            "account_id": "takemiya@test",
            "quorum": 5
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | ID of account to set quorum | already existent, account_name@domain
Quorum | number of signatories needed to be included with a transaction from this account | 0 < quorum < 10

### Validation

When quorum is set, it is checked if invariant of **size(signatories) >= quorum** holds.

Two cases:
Case 1. When transaction creator wants to set quorum for his/her account and he or she has permission CanRemoveSignatory
Case 2. Transaction creator was granted with CanRemoveSignatory permission to remove signatory from this account

## Subtract asset quantity

### Purpose

The purpose of _subtract asset quantity_ is the opposite of AddAssetQuantity commands — to decrease the number of assets on account of transaction creator. 

### Structure

```protobuf
message AddAssetQuantity {
    string account_id = 1;
    string asset_id = 2;
    Amount amount = 3;
}

message uint256 {
   uint64 first = 1;
   uint64 second = 2;
   uint64 third = 3;
   uint64 fourth = 4;
}

message Amount {
   uint256 value = 1;
   uint32 precision = 2;
}

```
```json
{
    "commands": [
        {
            "command_type": "SubtractAssetQuantity",
            "account_id": "test@test",
            "asset_id": "coin#test",
            "amount": {
                "value": string,
                "precision": int
            }
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id from which to subtract asset | account_name@domain
Asset ID  | id of the asset | asset#account
Amount  | positive amount of the asset to add | > 0

### Validation

1. Asset and account should exist
2. Added quantity precision should be equal to asset precision
3. Creator of transaction should have role which has permissions for subtraction of assets
4. Creator of transaction subtracts account quantity in his/her account only

## Transfer asset

### Purpose

The purpose of _transfer asset_ is to share assets within the account in peer network: in the way that source account transfers assets to target account.

### Structure

```protobuf
message TransferAsset {
    string src_account_id = 1;
    string dest_account_id = 2;
    string asset_id = 3;
    string description = 4;
    Amount amount = 5;
}
```
```json
{
    "commands": [
        {
            "command_type": "TransferAsset",
            "src_account_id": "takemiya@test",
            "dest_account_id": "nikolai@test",
            "asset_id": "coin#test",
            "description": "Salary payment",
            "amount": {
                "int_part": 20,
                "precision": 0
            }
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Source account ID | ID of account to withdraw asset from | already existent, account_name@domain
Destination account ID | ID of account to send asset at | already existent, account_name@domain
Asset ID | ID of asset to use | already existent, asset_name#domain
Description | Message to attach to transfer | No constraints
Amount | amount of asset to transfer | 0 < amount < max_uint256

### Validation

1. Source account has this asset in its AccountHasAsset relation
2. An amount is a positive number and asset precision is consistent with the asset definition
3. Source account has enough amount of asset to transfer and is not zero
4. Source account can transfer money, and destination account can receive money (their roles have these permissions)