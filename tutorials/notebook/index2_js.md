---
layout: default
title: index2.js
parent: Notebook
nav_order: 2
---

```javascript
const tupelo = require('tupelo-wasm-sdk');
const fs = require('fs');

const LOCAL_ID_PATH = './.notebook-identifiers';
const CHAIN_TREE_NOTE_PATH = 'notebook/notes';

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

async function readIdentifierFile() {
    console.log("reading identifiers")
    let raw = fs.readFileSync(LOCAL_ID_PATH);
    const identifiers = JSON.parse(raw);
    const keyBits = Buffer.from(identifiers.unsafePrivateKey, 'base64')
    const key = await tupelo.EcdsaKey.fromBytes(keyBits)

    const community = await tupelo.Community.getDefault()
    let tree
    try {
        const tip = await community.getTip(identifiers.chainId)
        console.log("found tree")
        tree = new tupelo.ChainTree({
            store: community.blockservice,
            tip: tip,
            key: key,
        })
    } catch(e) {
        if (e === 'not found') {
            tree = await tupelo.ChainTree.newEmptyTree(community.blockservice, key)
        } else {
            throw e
        }
    }

    return { tree: tree, key: key }
}

function idFileExists() {
    return fs.existsSync(LOCAL_ID_PATH);
}

function addTimestamp(note) {
    let ts = new Date().getTime().toString();
    return ts + '::' + note;
}

async function createNotebook() {
    console.log("creating notebook")
    let community = await tupelo.Community.getDefault();
    const key = await tupelo.EcdsaKey.generate()
    const tree = await tupelo.ChainTree.newEmptyTree(community.blockservice, key)
    let obj = await identifierObj(key, tree);
    return writeIdentifierFile(obj);
}

async function addNote(note) {
    if (!idFileExists()) {
        console.error("Error: you must register before you can record notes.");
        return;
    }

    let { tree } = await readIdentifierFile();
    console.log("resolving data on tree")
    const resp = await tree.resolveData(CHAIN_TREE_NOTE_PATH);
    let notes = resp.value,
        noteWithTs = addTimestamp(note);

    if (notes instanceof Array) {
        notes.push(noteWithTs);
    } else {
        notes = [noteWithTs];
    }

    console.log("saving new notes: ", notes)
    let c = await tupelo.Community.getDefault()
    await c.playTransactions(tree, [tupelo.setDataTransaction(CHAIN_TREE_NOTE_PATH, notes)])
}
```
