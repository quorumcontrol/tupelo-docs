---
layout: default
title: index1.js
parent: Notebook
nav_order: 2
---
```javascript
const tupelo = require('tupelo-wasm-sdk');
const fs = require('fs');

const LOCAL_ID_PATH = './.notebook-identifiers'; // <--- Specify the file to save to

async function identifierObj(key, chain) {
    return {
        unsafePrivateKey: Buffer.from(key.privateKey).toString('base64'),
        chainId: await chain.id()
    };
}

function writeIdentifierFile(configObj) {
    console.log("saving writeIdentifier: ", configObj)
    let data = JSON.stringify(configObj);
    fs.writeFileSync(LOCAL_ID_PATH, data);
}

async function createNotebook() {
    console.log("creating notebook")
    let community = await tupelo.Community.getDefault();
    const key = await tupelo.EcdsaKey.generate()
    const tree = await tupelo.ChainTree.newEmptyTree(community.blockservice, key)
    let obj = await identifierObj(key, tree);
    return writeIdentifierFile(obj);
}
```
