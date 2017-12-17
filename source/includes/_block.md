# Block structure

![Block](../images/block.png "Block structure")

In order to understand contents of the block better, this section tells about its parts:

### Outside payload
* hash — SHA3-512 hash of block protobuf payload
* signatures — signatures of peers, which voted for the block during consensus round

### Inside payload
* height — a quantity of blocks in the chain up to the block
* timestamp — unix time (in millis) of block forming by a peer
* body — transactions, which successfully passed validation and consensus step
* transactions quantity
* previous hash of block
