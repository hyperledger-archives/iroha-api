# Overview

Welcome to the Hyperledger Iroha API docs!

<iframe width="560" height="315" style="padding-left: 28px" src="https://www.youtube.com/embed/Hf_5j0b6TZM" frameborder="0" allowfullscreen></iframe>

This API allows users to send transactions to peer network, containing one or many commands to perform allowed actions in the system, and also make queries to know current state.
Users are authenticated by their keypair and account id, passed as the parameter with their transactions or queries.

The API is organized around protobuf format, as Iroha is using gRPC on transport level. Iroha CLI and blockstore works with JSON format to provide developer-friendly experience for community members.  

To try out a basic API request, do the following:

1. Run the system (`irohad` daemon) on a single node
2. Run iroha-cli in interactive mode
3. Select a necessary action to perform
4. Send formed protobuf via gRPC call
5. Get a response from the system 

