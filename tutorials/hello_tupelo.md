---
layout: default
title: Hello Tupelo
parent: Tutorials
nav_order: 1
---

# Hello Tupelo
{: .fs-9 }

## Prerequisites
1. [Start the Tupelo RPC server](./rpc_server.md).
2. Run `mkdir tupelo && cd tupelo` at a shell prompt.
3. Install the client with `npm install tupelo-client`
4. Launch a Node.js repl with `node`.

## At the Node REPL
```javascript
// Load the tupelo client library
> const tupelo = require('tupelo-client');

// Connect to the RPC server
> const client = tupelo.connect('localhost:50051', { walletName: 'billfold', passPhrase: 'this is a secret' });

// Register the wallet on the RPC server
> var registration = client.register();
> registration.then((result) => { console.log("--- success: wallet registered")}, (err) => { console.log("--- error: "+err) });

// Generate a new keypair
var keyAddr;
client.generateKey().then((result) => { console.log("--- success!"); keyAddr = result.keyAddr }, (err) => { console.log("--- error: "+err)});

// Create a new chain tree
var chainId;
client.createChainTree(keyAddr).then((result) => { console.log("--- success!"); chainId = result.chainId }, (err) => { console.log("--- error: "+err) });

// Write some data to the tree
const testPath = "my/data/path";
client.setData(chainId, keyAddr, testPath, "Hello World!").then((result) => { console.log("--- success!") }, (err) => { console.log("--- error: "+err) });

// Read the data back
client.resolve(chainId, testPath).then((result) => { console.log("--- success!"); console.log(result.data.toString()) }, (err) => { console.log("--- error: "+err) });
```

Please take a look at the [Tupelo JavaScript API Documentation](https://quorumcontrol.github.io/tupelo.js/) for more information on what you can do with the API.
