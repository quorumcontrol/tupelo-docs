---
layout: default
title: index.js
parent: Notebook
nav_order: 2
---

```javascript
const tupelo = require('tupelo-client');
const fs = require('fs');
const yargs = require('yargs');

const LOCAL_ID_PATH = './.notebook-identifiers';
const CHAIN_TREE_NOTE_PATH='notebook/notes';

function connect(creds) {
  return tupelo.connect('localhost:50051', creds);
}

function identifierObj(key, chain) {
  return {
    keyAddr: key,
    chainId: chain
  };
}

function writeIdentifierFile(configObj) {
  let data = JSON.stringify(configObj);
  fs.writeFileSync(LOCAL_ID_PATH, data);
}

function readIdentifierFile() {
  let raw = fs.readFileSync(LOCAL_ID_PATH);
  return JSON.parse(raw);
}

function idFileExists() {
  return fs.existsSync(LOCAL_ID_PATH);
}

function createNotebook(creds) {
  let client = connect(creds);
  var keyAddr, chainId;

  client.register()
    .then(function(registerResult){
      return client.generateKey();
    }, function(err) {
      console.log("Error registering wallet.");
      console.log(err.details);
    }).then(function(generateKeyResult) {
      keyAddr = generateKeyResult.keyAddr;
      return client.createChainTree(keyAddr);
    }, function(err) {
      console.log("Error generating key.");
      console.log(err.details);
    }).then(function(createChainResponse) {
      chainId = createChainResponse.chainId;
      console.log("Saving registration.");
      let obj = identifierObj(keyAddr, chainId);
      return writeIdentifierFile(obj);
    }, function(err) {
      console.log("Error creating chain tree.");
      console.log(err.details);
    });
}

function addTimestamp(note) {
  let ts = new Date().getTime().toString();
  return ts + '::' + note;
}

function addNote(creds, note) {
  if (!idFileExists()) {
    console.log("Error: you must register before you can record notes.");

    return;
  }

  let client = connect(creds),
      identifiers = readIdentifierFile();

  client.resolveData(identifiers.chainId, CHAIN_TREE_NOTE_PATH)
    .then(function(resp) {
    let notes = resp.data[0],
        noteWithTs = addTimestamp(note);

      if (notes instanceof Array) {
        notes.push(noteWithTs);
      } else {
        notes = [noteWithTs];
      }

      return client.setData(identifiers.chainId,
                            identifiers.keyAddr,
                            CHAIN_TREE_NOTE_PATH,
                            notes);
  }, function(err) {
      console.log('Error reading notes: ' + err);
  });
}

function showNotes(creds) {
  if (!idFileExists()) {
    console.log("Error: you must register before you can print notes.");

    return;
  }

  let identifiers = readIdentifierFile();
  let client = connect(creds);
  let path = CHAIN_TREE_NOTE_PATH;

  client.resolveData(identifiers.chainId, CHAIN_TREE_NOTE_PATH)
    .then(function(resp) {
      let notes = resp.data;

      if (notes instanceof Array) {
        console.log('----Notes----');
        notes.forEach(function(note) {
          console.log(note);
        });
      } else {
        console.log('----No Notes-----');
      }
    }, function(err) {
      console.log('Error fetching tally: '  + err);
    });
}

yargs.command('register [name] [passphrase]', 'Register a new notebook chain tree', (yargs) => {
  yargs.positional('name', {
    describe: 'Name of the wallet to save the notes chain tree.'
  }).positional('passphrase', {
    describe: 'Wallet passphrase.'
  });
}, (argv) => {
  let creds = {
    walletName: argv.name,
    passPhrase: argv.passphrase
  };

  createNotebook(creds);

}).command('add-note [name] [passphrase]', 'Save a note', (yargs) => {
  yargs.positional('name', {
    describe: 'Name of the wallet where the notes chain tree is saved.'
  }).positional('passphrase', {
    describe: 'Wallet passphrase.'
  }).describe('n', 'Save a note')
    .alias('n', 'note')
    .demand('n');
}, (argv) => {
  let creds = {
    walletName: argv.name,
    passPhrase: argv.passphrase
  };

  addNote(creds, argv.n);

}).command('print-notes [name] [passphrase]', 'Print saved notes', (yargs) => {
  yargs.positional('name', {
    describe: 'Name of the wallet where the notes chain tree is saved.'
  }).positional('passphrase', {
    describe: 'Wallet passphrase.'
  });
}, (argv) => {
  let creds = {
    walletName: argv.name,
    passPhrase: argv.passphrase
  };

  showNotes(creds);
}).argv;
```
