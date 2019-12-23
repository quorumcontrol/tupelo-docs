---
layout: default
title: Notebook
parent: Tutorials
nav_order: 2
---

# Building a Notebook
We're going to build a command-line timestamped note-taking application using
the [Tupelo WASM SDK](https://quorumcontrol.github.io/tupelo-js-sdk "Tupelo Javascript SDK").
When we're done, we'll be able to use this app to save small timestamped notes
over time, and later display all the notes in order.

## Getting Started
Our notebook will be a [Node.js](https://nodejs.org/en/ "Node.js") application.
We'll manage its dependencies with [NPM](https://www.npmjs.com/ "NPM"), so make
sure you have already installed it before going further.

Next, we'll create and enter the directory that will hold our notebook
application's code, and then initialize a new NPM application with `npm init`.
The `npm init` defaults are fine for our purposes, so we'll use the `-y` option
to skip any confirmation questions.

```bash
# Create the 'notebook' application directory
mkdir notebook

# Change to the application directory
cd notebook

# Initialize a new npm application
npm init -y
```

This gives us a new directory with a `package.json` skeleton to start with.

## The Tupelo WASM SDK
Before we build anything, we need to add the Tupelo WASM SDK to our application's
dependency set. Edit the new `package.json` file in your favorite editor and add a
new dependency set containing only the `tupelo-wasm-sdk` library:

In file `notebook/package.json`:
```json
{
  "name": "notebook",
  ...
  "dependencies": {
      "tupelo-wasm-sdk": "latest"
  },
}
```

Next, run `npm install` from the application directory to install the dependency
to your local cache.

After installing the `tupelo-wasm-sdk` dependency, make a new file called
`index.js` with your favorite editor to hold all the application code. At the
top of that file, require the Tupelo wasm sdk so we can use it later.

In file `notebook/index.js`:
```javascript
const sdk = require('tupelo-wasm-sdk');
```

## Creating a New Notebook
Our application will organize notes in _notebooks_, and we'll store the data
associated with a particular notebook in its own, unique chain tree. Before we
can start saving notes, we need to connect to a service to keep track of our Chaintrees,
create some keys and then create a ChainTree to store our notes.

Let's build a `createNotebook()` function, step by step, to do those things.

First we will connect to the community service and the Tupelo TestNet, in `notebook/index.js`:
```javascript
async function createNotebook() {
    console.log("creating notebook")
    let community = await sdk.Community.getDefault();
}
```

### Generate Keys

Next, we will generate a new public/private keypair for the 'user' of our wasm app in
the `createNotebook()` function.

In file `notebook/index.js`:
```javascript
async function createNotebook() {
    console.log("creating notebook")
    let community = await sdk.Community.getDefault();
    const key = await sdk.EcdsaKey.generate()
}
```

### Store Identifiers

We will need a way to store the information we need locally so we can keep track of these
identifiers between `createNotebook()` invocations.  We could use a database for this in a
full-fledged production app, but an external file is enough to serve our purposes.
The `fs` module handles file i/o in Node.js so we will add a filesystem require for that:
```javascript
const sdk = require('tupelo-wasm-sdk');
const fs = require('fs');
```

Then we will create a functions to compose our identifier object and write it the filesystem
so it can be accessed later:

```javascript
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
```

Finally we will finish our our notebook creation by creating a new empty ChainTree
and then saving it by calling our two new functions to compose and store our key and
the identifier for our notebook ChainTree.

```javascript
async function createNotebook() {
    console.log("creating notebook")
    let community = await sdk.Community.getDefault();
    const key = await sdk.EcdsaKey.generate()
    const tree = await sdk.ChainTree.newEmptyTree(community.blockservice, key)
    let obj = await identifierObj(key, tree);
    return writeIdentifierFile(obj);
}
```

### Testing our Notebook creation

Now let's try out what we have so far. Start a node repl in your project
directory with the `node` command. Then, from the node prompt run

```javascript
> .load index.js
> createNotebook();
```

This sequence of commands will first load our `index.js` file as if we'd typed
each line into the repl and then runs our `createNotebook()` function.

You should see a confirmation that we are saving our writeIdentivier with a PrivateKey
and a chainID in the console.  As long as you do see those '.exit' the node repl session.
There should be a .notebook-identifiers file with our key and chain ID in it.

--------------------

### Adding a note to our Notebook

Now that we have a notebook its time to start writing our notes into it.

Now lets start building our function for actually adding a note at that path in the
Chaintree.  We will want to start by creating an addNote function accepting an argument
of the value of the new note to be added.

```javascript
async function addNote(note) {

}
```

### Retrieving our key and notebook

Now before we write our new note we actually need to make sure we have the identifiers
we need.  Lets create a function to grab the information we stored in our Identifier
file we created during notebook creation.  

We start readIdentifierFile by opening the file at our LOCAL_ID_PATH
and then grab and parse the key we stored there translating it into the appropriate form.

```javascript
async function readIdentifierFile() {
    let raw = fs.readFileSync(LOCAL_ID_PATH);
    const identifiers = JSON.parse(raw);
    const keyBits = Buffer.from(identifiers.unsafePrivateKey, 'base64')
    const key = await sdk.EcdsaKey.fromBytes(keyBits)
}
```

Then we connect to the community service where we had put our notebook Chaintree:
```javascript
...
const community = await sdk.Community.getDefault()
...
```

We then grab the "tip" of our Chaintree which represents the latest version of it from the
community service.  We need to pass the chain ID from the identifier file to grab the
notebook we own so that we can update it.  Once we have found the tree we load it up.

``` javascript
...
let tree
try {
    const tip = await community.getTip(identifiers.chainId)
    console.log("found tree")
    tree = new sdk.ChainTree({
        store: community.blockservice,
        tip: tip,
        key: key,
    })
}
...
```

We will want to catch an error and create a new empty Chaintree if we could not
find our existing one for some reason.

``` javascript
} catch(e) {
    if (e === 'not found') {
        tree = await sdk.ChainTree.newEmptyTree(community.blockservice, key)
    } else {
        throw e
    }
}
...
```

Upon success, we return the notebook chaintree and the key we have retrieved.

``` javascript
  return { tree: tree, key: key }
}
```

This results in the following full readIdentifierFile function:

``` javascript
async function readIdentifierFile() {
    console.log("reading identifiers")
    let raw = fs.readFileSync(LOCAL_ID_PATH);
    const identifiers = JSON.parse(raw);
    const keyBits = Buffer.from(identifiers.unsafePrivateKey, 'base64')
    const key = await sdk.EcdsaKey.fromBytes(keyBits)

    const community = await sdk.Community.getDefault()
    let tree
    try {
        const tip = await community.getTip(identifiers.chainId)
        console.log("found tree")
        tree = new sdk.ChainTree({
            store: community.blockservice,
            tip: tip,
            key: key,
        })
    } catch(e) {
        if (e === 'not found') {
            tree = await sdk.ChainTree.newEmptyTree(community.blockservice, key)
        } else {
            throw e
        }
    }

    return { tree: tree, key: key }
}
```

### Writing the note

Now that we have a way to retrieve our notebook ChainTree and key we can proceed
towards inserting our new note.

We will need to decide where in our ChainTree to put the data we are creating.  

In file `notebook/index.js` we will add a constant to store that.
```javascript
...
const CHAIN_TREE_NOTE_PATH = 'notebook/notes';
...
```

To build the addNote function we will want to start by grabbing our identifiers and then
whatever notes already exist. Because ChainTrees are so flexible this data can be of
nearly any type.  We will store our notes in an array at that location.

``` javascript
async function addNote(note) {

    let { tree } = await readIdentifierFile(); // Using our new function to retrieve

    const resp = await tree.resolveData(CHAIN_TREE_NOTE_PATH);
    let notes = resp.value,
        noteWithTs = addTimestamp(note);

    if (notes instanceof Array) {
        notes.push(noteWithTs);
    } else {
        notes = [noteWithTs];
    }

}
```

You will note that along with our text values we have decided to add a timestamp as well.  
This will provide additional immutable information to our notebook to be signed by
the Tupelo Network appending when our notes content was entered.

We will need to add a simple function to support that timestamping.

``` javascript
function addTimestamp(note) {
    let ts = new Date().getTime().toString();
    return ts + '::' + note;
}
```

The last step we need to take in our addNote function is to actually submit the
information to be signed!  So to the end of addNote() we will do just that.

``` javascript
    ...
    console.log("saving new notes: ", notes)
    let c = await sdk.Community.getDefault()
    await c.playTransactions(tree, [sdk.setDataTransaction(CHAIN_TREE_NOTE_PATH, notes)])
}
```

### Making sure the notebook exists before we write

We want to make sure that everything is done in the right order so we will create a
small function to confirm we have an id file created:

``` javascript
function idFileExists() {
    return fs.existsSync(LOCAL_ID_PATH);
}
```
Then we will want to call that at the very beginning of our addNote() function.  If
there is no ID file we will warn the user and end there.

``` javascript
if (!idFileExists()) {
    console.error("Error: you must register before you can record notes.");
    return;
}
```

In the end this leave us with this [`index.js` file](/tutorials/notebook/index2_js).


### Testing our add note functionality

Now's a good time to test our `addNote()` function. Start another node repl session in
the notebook directory with the `node` command, then run these next few commands:

```javascript
> .load index.js
> createNotebook();
> addNote("Super Awesome Note of Consequence.");
```

Assuming everything worked, you should see a "Saving Registration" message at
the node repl after the createNotebook.

In response to "addNote" you should get a confirmation
"saving new notes: [ '1577129740043::Super Awesome Note of Consequence.' ]"
The first time you save a note you should also get confirmation that "nothing was resolvable"
because its the first time any data was placed there.  We can add additional notes
and the array of notes will grow but we will not see that message again.

Assuming everything worked terminate the node repl session with the `.exit` command.

If you are encountering problems you can check your index.js file against this
[`index.js` file](/tutorials/notebook/index2_js).

### Displaying our Notes

Since we can save and sign notes to our chain tree, let's print out all the
notes we've recorded so far. We'll write a `showNotes()` function that fetches
the saved notes using the functions we have already created and prints each one
to the console. Just like our function to save notes, we will also make sure that
the user has already created a wallet and saved some notes.

In many ways showing notes is a more straight version of adding notes since the
first step to adding notes was retriving our existing ones.

First we want to make sure the user has registered.
Then we populate our ChainTree of notes locally into the tree variable.
It is then a simple matter of resolving the data from our CHAIN_TREE_NOTE_PATH
and finally cycling through each value in the array printing them to the console.

```javascript
async function showNotes() {
    if (!idFileExists()) {
        console.error("Error: you must register before you can print notes.");
        return;
    }

    let { tree } = await readIdentifierFile();
    let resp = await tree.resolveData(CHAIN_TREE_NOTE_PATH)
    let notes = resp.value

    if (notes instanceof Array) {
        console.log('----Notes----');
        notes.forEach(function (note) {
            console.log(note);
        });
    } else {
        console.log('----No Notes-----');
    }
}
```
If there are no notes we simply output that.

### Creating a command line Interface

Now that we have all the background functions we need to manage notes with our
application, let's add a command line interface to tie everything together.
We'll use the [Yargs](https://github.com/yargs/yargs "Yargs") library for the
CLI, so let's add it as a dependency to our `package.json` and require it at the
top of our `index.js` file.


In file `notebook/package.json`:
```javascript
{
    "name": "notebook",
    ...
    "dependencies": {
        "tupelo-client": "^0.4.0",
        "yargs": "^12.0.2"
    }
}
```

Next run `npm install` to get yargs loaded.

In file `notebook/index.js`:
```javascript
const tupelo = require('tupelo-client');
const fs = require('fs');
const yargs = require('yargs');
...
```

Next we'll follow the `yargs` documentation to define 3 commands: `register`,
`add-note`, and `print-notes`.

In file `notebook/index.js`:
```javascript
yargs.command('register', 'Register a new notebook chain tree', (yargs) => {
}, async (argv) => {
    await createNotebook();
    process.exit(0)
}).command('add-note', 'Save a note', (yargs) => {
    yargs.describe('n', 'Save a note')
        .alias('n', 'note')
        .demand('n');
}, async (argv) => {
    await addNote(argv.n);
    process.exit(0)
}).command('print-notes', 'Print saved notes', (yargs) => {
}, async (argv) => {
    await showNotes();
    process.exit(0)
}).argv;
```

## Finishing Up

Now we've built a command line notebook that, when invoked with `node`, can
record timestamped notes into a chain tree, print them out later, while the
development notary group validates each note as we save them.

Now you can run `node ./index.js register` to register a new notebook,
`node ./index.js add-note -n <note>` to save a note, and finally,
`node ./index.js print-notes` to print all the saved notes.

Be sure to take a look at the final
[`package.json` file](/tutorials/notebook/package_json) and the final
[`index.js` file](/tutorials/notebook/index3_js).
