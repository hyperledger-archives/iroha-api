# Overview

Welcome to the Hyperledger Iroha API docs!

## Elevator pitch for Iroha

The system is a distributed ledger platform, capable of storing all history of state-changing actions (called transactions) in immutable linear history (called blockchain). The system is intended to be used in enterprise or between untrusted agents as **private chain platform**, but not as a public chain — key difference is that not everyone is allowed to store the whole history. A subset of users can only query some data (perform a query) if their accounts have required permissions, while others maintain a peer (Iroha is a peer network), which stores history. 

Reliability and security are provided with cryptographic capabilities and **Byzantine Fault Tolerant** consensus algorithm. The system is targeted to environments, where client applications are very diverse (desktop, various mobile platforms), and peer hardware varies from embedded systems to enterprise-class servers.

## Purpose of the docs you are reading

The documentation describes the endpoints for users to send transactions to peer network, containing one or many commands to perform allowed actions in the system, and also make queries to know the current state. Also the documentation contains explanation how to build the system, how to run it and other applicable tutorials. 

The API is organized around protobuf format, as Iroha is using gRPC in transport level. Iroha CLI and block store use JSON format to provide developer-friendly experience for contributors. Feel free to check both sections for protobuf and JSON while exploring docs.

## Try it yourself
 
To try out a basic API functionality, do the following:

<aside class="notice">
In case you have troubles understanding some specific terminology used — refer to the <a href="#glossary">glossary</a>.
</aside>

1. Run the system (`irohad` daemon) on a single node.
2. Run iroha-cli in interactive mode. Command Line Interface app is a client, showing the capabilities of the system.
3. Select a necessary action to perform. As you created an initial configuration in genesis block — use the account from genesis block to send transaction or make a query.
4. When you fill all the necessary details for commands and formed a transaction; or if you formed a query — you are ready to send it to Iroha peer. Tell the network address and port (by default it is 50051).

<aside class="success">
If everything went well and you were successful to form transaction or query with valid fields — you will get either transaction hash, which is used for querying transaction state, or query response object (see query subsection for more details).
</aside>
