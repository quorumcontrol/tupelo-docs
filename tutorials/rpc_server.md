---
layout: default
title: RPC Server
parent: Tutorials
nav_order: 3
---

# Running the Tupelo RPC server
The Node.js client cannot directly manage chain trees or connect to the notary
group, so node applications must instead proxy through an RPC server to work
with Tupelo.

To install the server, first contact us to get a Tupelo binary for your platform
and save it within your command `PATH` variable. You can download the latest
 [Tupelo v0.0.6 executable here](rpc_server/tupelo-v0.0.6.zip). 
If you do not wish to save the
binary to a directory in your `PATH`, you can still execute it with the fully
qualified or relative path to your chosen location for the binary.

After successfully installing the binary, you can run the RPC server by invoking
`tupelo` (or the full or relative path to your chosen install location if you
did not install Tupelo in a `PATH` directory) along with the necessary options.
Production applications will usually connect to an independent notary group, but
you can also choose to run and connect to a local notary group to make working
in development easier.

## Connecting the RPC Server to a Local Notary Group
To start the RPC server and notary group for local development, run:
```bash
tupelo rpc-server
```

This invocation will start a 3 signer local notary group after first generating
random keypairs for the group to use. After that, the RPC server will start,
bind itself to the local notary group, and listen on port 50051.
