---
layout: default
title: Hello Tupelo
parent: Tutorials
nav_order: 1
---

# Hello Tupelo
{: .fs-9 }

## Shell

1. Start the shell
```bash
$ ./tupelo shell -w billfold
```

2. Create a new, password protected wallet (you only need to do this once)
```
>>> create-wallet
Creating wallet:  billfold
Please enter a new passphrase:
Please confirm your passphrase by entering it again:
Thank you for confirming your password.
```

3. Start a new session
```
>>> start-session
Passphrase:
Starting session
```

4. Generate a new key pair
```
>>> create-key
\<key address>
```

5. Create a new chain tree
```
>>> create-chain <key address>
chain-id: <chain tree id>
```

6. Write some data to the tree
```
>>> set-data <chain tree id> <key address> my/data/path "hello tupelo"
new tip: <tip hash>
```

7. Read the data back
```
>>> resolve <chain tree id> my/data/path
data: hello tupelo
remaining path: []
```

## Node.js
### Prerequisites
1. [Start the Tupelo RPC server](/tutorials/rpc-server).
2. Run `mkdir tupelo && cd tupelo` at a shell prompt.
3. Install the client with `npm install tupelo-client`
4. Launch a Node.js repl with `node`.

### At the Node REPL
```javascript
// Load the tupelo client library
> const tupelo = require('tupelo-client');

// Connect to the RPC server
> const client = tupelo.connect('localhost:50051', { walletName: 'billfold', passPhrase: 'this is a secret' });

// Register the wallet on the RPC server
> var registration = client.register();
> registration.then((result) => { console.log("--- success: wallet registered")}, (err) => { console.log("--- error: "+err) });

// Generate a new keypair
> var keyAddr;
> client.generateKey().then((result) => { console.log("--- success!"); keyAddr = result.keyAddr }, (err) => { console.log("--- error: "+err)});

// Create a new chain tree
> var chainId;
> client.createChainTree(keyAddr).then((result) => { console.log("--- success!"); chainId = result.chainId }, (err) => { console.log("--- error: "+err) });

// Write some data to the tree
> const testPath = "my/data/path";
> client.setData(chainId, keyAddr, testPath, "Hello World!").then((result) => { console.log("--- success!") }, (err) => { console.log("--- error: "+err) });

// Read the data back
> client.resolve(chainId, testPath).then((result) => { console.log("--- success!"); console.log(result.data.toString()) }, (err) => { console.log("--- error: "+err) });
```

Please take a look at the [Tupelo JavaScript API Documentation](https://quorumcontrol.github.io/tupelo.js/) for more information on what you can do with the API. If you'd like to build an app on Tupelo, then take a look at our [Notebook tutorial](/tutorials/notebook).
