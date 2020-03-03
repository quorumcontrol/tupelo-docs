---
layout: default
title: Hello from Node
parent: Tutorials
nav_order: 1
---

# Hello Decentralized World -- from Node

## Quickstart
To show how easy Tupelo is to use we are going to create our first ChainTree and then add
data to it ("Hello World" of course) and have it signed by the Tupelo TestNet.  

We are going to accomplish this by using the Tupelo WASM SDK and a node repl session.

First you will need to use [npm](https://www.npmjs.com/get-npm) to install two dependencies.

```
npm install tupelo-wasm-sdk@0.6.0-rc3
npm install multicodec@0.5.6
```

Next we will hop into a [node](https://nodejs.org/en/download/) session (with a flag enabling promises to make our life easier).

```
node --experimental-repl-await
```

Next follow these commands in the node session (ignoring 'undefined' responses as you go):

```
> const sdk = require('tupelo-wasm-sdk')
> const community = await sdk.Community.getDefault()
> const key = await sdk.EcdsaKey.generate()
> const tree = await sdk.ChainTree.newEmptyTree(community.blockservice, key)
> let resp = await community.playTransactions(tree, [sdk.setDataTransaction( "path", "Hello Decentralized World!")])
```

Congratulations!  
You have successfully written to a ChainTree and your request was verified by the Tupelo network.

Please note the PEREFORMANCE of Tupelo during that session.  How long did it take to create a ChainTree?
How long did it take to get the data update transaction signed?  The transaction was submitted, the
network came to consensus on it and your transaction was confirmed in that time. If it was more than a second or two something was very wrong.

Don't believe it could be that easy?  
As a double check, lets do two extra steps to confirm our ChainTree is there.
Continuing from the session above lets fetch the data to check our value and then show our CID:

```
> await tree.resolveData("path")
> await tree.id()
```

## Commentary
Read below if you'd like a (brief) explanation of what each of the above steps does.

1. Setup a connection to the default community and Tupelo TestNet
```
> const community = await sdk.Community.getDefault() 
```

1. Generate a new public/private keypair 
```
> const key = await sdk.EcdsaKey.generate() 
```

3. Create a new empty tree with the new keypair
```
> const tree = await sdk.ChainTree.newEmptyTree(community.blockservice, key) 
```

4. Play a transaction on the TestNet -- Essentially add data to your Chaintree and get it signed by the network.
```
> let resp = await community.playTransactions(tree, [sdk.setDataTransaction("path", "Hello Decentralized World!")])
```

If you want to read more about each of these commands you can check out the [WASM SDK API](https://quorumcontrol.github.io/tupelo-wasm-sdk/docs/tupelo-wasm-sdk.html)

If you are ready to move on to building a small application to further get to know the Tupelo DLT, try our [notebook tutorial](/tutorials/notebook) and build a simple app.
