# Commands

## Add asset quantity

### Purpose

Purpose of _add asset quantity_ is to increase amount of assets on some account. 

### Structure

```protobuf
message AddAssetQuantity {
	string account_id = 1;
	string asset_id = 2;
	Amount amount = 3;
}
```
```json
{
    "commands": [
        {
            "command_type": "AddAssetQuantity",
            "account_id": "test@test",
            "asset_id": "coin#test",
            "amount": object
        }
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account ID | account id in which to add asset | username@domain
Asset ID  | id of the asset | asset#account
Amount  | positive amount of the asset to add | > 0

### Validation

?

## Add peer

### Purpose

Purpose of _add peer_ is to write into ledger fact of peer addition into the peer network.

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
Peer key | peer public key, which will be used in consensus algorithm to sign-off vote, commit, reject messages | **?**


### Validation

?


## Add signatory

### Purpose

Purpose of _add signatory_ is to add identity to the account. It can be the public key of another device or public key of another user.

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
Account ID | Account in which to add signatory (username@domain) | username@domain
Public key | Signatory to add to account | ed25519 public key

### Validation

?

## Create asset

### Purpose

Purpose of _сreate asset_ is to create new type of asset, specific for a domain.

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
Asset name | domain-unique name for asset | **?**
Domain ID | target domain to make relation with | should be created before the asset, length — **?**
Precision | number of digits after comma/dot | 0 <= precision <= uint32 max

### Validation

?

## Create account

### Purpose

Purpose of _create account_ is to make new entity similar to wallet, which stores assets and identities.

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
            "account_name": "takemiya", 
            "domain_id": "test", 
            "main_pubkey": string(64)
        } 
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Account name | domain-unique name for account | **?**
Domain ID | target domain to make relation with | should be created before the account, length is **?**
Main pubkey| first piblic key to add into the account | ed25519 public key

### Validation

?

## Create domain

### Purpose

Purpose of _create domain_ is to make new domain in Iroha network.

### Structure

<aside class="notice">
Just in case it is not working — it was suggested to rename domain_name field into domain_id on 24th of September for consistency across the project.
</aside>

<aside class="notice">
Also, regarding recent API changes, we are going to add _default role_ in account so that any created account in this domain has this role by default.
</aside>

```protobuf
message CreateDomain {
 string domain_name = 1;
}
```
```json
{
    "commands": [
        {
            "command_type": "CreateDomain", 
            "domain_name": "test2"
        } 
    ], …
}
```

Field | Description | Constraint
-------------- | -------------- | --------------
Domain ID | ID for created domain | unique, regex is **?**

### Validation

* _domain id_ is unique and valid against regex
* Account, who sends this command in transaction, has permission to create domain

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
Account ID | ID of account to delete signatory from | already existent, regex is **?**

### Validation

When signatory is deleted, we should check if invariant of **size(signatories) >= quorum** holds

## Set account quorum

### Purpose

Purpose of _set account quorum_ is to set the number of signatories needed to confirm the identity of person, sending the transaction or is to set amount of people to agree on transaction contents.

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
Account ID | ID of account to set quorum | already existent, regex is **?**
Quorum | number of signatories needed to be included with a transaction from this account | # less than 

### Validation

When quorum is set, we should check if invariant of **size(signatories) >= quorum** holds

## Set account quorum

### Purpose

Purpose of _set account quorum_ is to set the number of signatories needed to confirm the identity of person, sending the transaction or is to set amount of people to agree on transaction contents.

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
Account ID | ID of account to set quorum | already existent, regex is **?**
Quorum | number of signatories needed to be included with a transaction from this account | # less than 

### Validation

When quorum is set, we should check if invariant of **size(signatories) >= quorum** holds

## Transfer asset

### Purpose

Purpose of _transfer asset_ is to share assets across the network, so that source account can transfer assets to target account.

### Structure

```protobuf
message TransferAsset {
 string src_account_id = 1;
 string dest_account_id = 2;
 string asset_id = 3;
 Amount amount = 4;
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
Source account ID | ID of account to withdraw asset from | already existent, regex is **?**
Destination account ID | ID of account to send asset at | already existent
Asset ID | ID of asset to use | already existent
Amount | amount of asset to transfer | **?** 

### Validation

?

## Subtract asset quantity

### Purpose

Purpose of _subtract asset quantity_ is to decrease amount of asset, belonging to an account.

### Structure

<aside class="warning">
This command is not implemented currently
</aside>

Field | Description | Constraint
-------------- | -------------- | --------------
? | ? | ?

### Validation

?

## Append role

### Purpose

Purpose of _append role_ is to promote an account to some created role in the system, where role is a set or permissions account has to perform an action (command or query).

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
Account ID | id or account to append role to | already existent, regex is **?**
Role name | name of already created role | already existent
Amount | amount of asset to transfer | **?** 

### Validation

?

## Create role

### Purpose

Purpose of _create role_ is to create new role in the system from the set of permissions.
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
Role name | name of role to create | regex?
Permissions | array of already existent permissions | **?** 

### Validation

?

## Grant permission

### Purpose

Purpose of _grant permission_ is to give another account in the system rights to perform actions over account of transaction sender (give someone right to do something with my account).

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
Account ID | id of account whom rights are granted | should be existent in the system
Permission name | name of granted permission | **?** 

### Validation

?

## Revoke permission

### Purpose

Purpose of _revoke permission_ is to revoke or dismiss given grant permission to another account in the netw

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
Account ID | id of account whom rights were granted | should be existent in the system
Permission name | name of revoked permission | **?** 

### Validation

?