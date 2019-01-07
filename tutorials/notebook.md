---
layout: default
title: Notebook
parent: Tutorials
nav_order: 2
---

# Building a Notebook
We're going to build a command-line timestamped note-taking application using
the [Tupelo.js](https://github.com/QuorumControl/tupelo.js "Tupelo.js") API.
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

## Running the Tupelo RPC server
The Node.js client cannot directly manage chain trees or connect to the notary
group, so node applications must instead proxy through an RPC server to work
with Tupelo.

To install the server, first contact us to get a Tupelo binary for your platform
and save it within your command `PATH` variable. If you do not wish to save the
binary in your `PATH`, you can still execute it with the fully qualified or
relative path to your chosen location for the binary.

After successfully installing the binary, you can run the RPC server by invoking
`tupelo` (or the path to your chosen install location) along with the necessary
options. Production applications will usually connect to an independent notary
group, but you can also choose to run and connect to a local notary group to
make working in development easier.

### Connecting the RPC Server to a Local Notary Group
To start the RPC server and notary group for local development, run:
```bash
tupelo rpc-server --local-network 3
```

This will start a 3 signer local notary group after first generating three
random keypairs for the group to use. Then it will start the RPC server and bind
it to the local notary group.

## Installing the Tupelo Client API
Before we build anything, we need to add the Tupelo client to our application's
dependency set. Edit the new `package.json` file in your favorite editor and add
a new dependency set containing only the `tupelo-client` library:

In file `notebook/package.json`:
```json
{
    "name": "notebook",
    ...
    "dependencies": {
        "tupelo-client": "^0.0.2-alpha1"
    }
}
```

Next, run `npm install` from the application directory to install the dependency
to your local cache.

After installing the `tupelo-client` dependency, make a new file called
`index.js` with your favorite editor to hold all the application code. At the
top of that file, require the Tupelo client library so we can use it later.

In file `notebook/index.js`:
```javascript
const tupelo = require('tupelo-client');
```

## Creating a New Notebook
Our application will organize notes in _notebooks_, and we'll store the data
associated with a particular notebook in its own, unique chain tree. Before we
can start saving notes, we need to register a wallet at the RPC server to store
the keys used to access the chain tree, create a new key, and create the chain
tree we'll use to store our notes. Let's build a `createNotebook()` function,
step by step, to do all those things.

### Connecting to the RPC Server
The Tupelo RPC server listens for connections on a specific port of the host its
running on after it starts. Since we did not specify the port [when we started
the server](#running-the-tupelo-rpc-server), our server is listening on "50051",
which is the default port. We will also need to supply a set of wallet
credentials to establish every connection to the RPC server. The credentials are
in an object with `walletName` and `passPhrase` keys.

Let's make a `connect` function that accepts wallet credential objects and
establishes a connection to the running development server.

In file `notebook/index.js`:
```javascript
function connect(creds) {
  return tupelo.connect('localhost:50051', creds);
}
```

Next, let's start building the `createNotebook()` function by establishing a
connection to the RPC server

In file `notebook/index.js`:
```javascript
function createNotebook(creds) {
  let client = connect(creds);
}
```

### Registering a New Wallet
In order to create and manipulate a chain tree in Tupelo, you must first create
or control the key associated with that chain tree. The RPC server manages those
keys in different wallets, but the client has to create the wallet and set its
credentials first. The `register()` method of the tupelo client returns a
promise that, when fulfilled, either contains the result of creating a new
wallet with the client's credentials, or an error if the wallet already
exists or there's another error during creation. Let's extend our
`createNotebook()` function to register a new wallet.

In file `notebook/index.js`:
```javascript
function createNotebook(creds) {
  let client = connect(creds);

  client.register()
    .then(function(registerResult){
      console.log("Success!");
    }, function(err) {
      console.log("Error registering wallet.");
      console.log(err);
    });
}
```

Upon success, we just print to the console that the action succeeded. In case an
error happened, we print the error to the console as well.

### Creating a New Key
After registering a new wallet, we'll need to create a new key using the
`generateKey()` client API method. Since that method, as all the other Tupelo
client methods, also returns a promise, we'll chain it to the `register()`
success case instead of printing to the console.

If the `generateKey()` request is successful, then the server will return a
response that contains the new key's public address under the `keyAddr` key.
We'll need to keep track of this key address for later, so we'll also declare a
variable to save it in.

In file `notebook/index.js`:
```javascript
function createNotebook(creds) {
  let client = connect(creds);
  var keyAddr;

  client.register()
    .then(function(registerResult){
      return client.generateKey()
    }, function(err) {
      console.log("Error registering wallet.");
      console.log(err);
    }).then(function(generateKeyResult) {
      keyAddr = generateKeyResult.keyAddr;
    }, function(err) {
      console.log("Error generating key.");
      console.log(err);
    });
}
```

### Creating a New Chain Tree
Once we've created a key, we can use it to create a new chain tree to store our
notebook data with the `createChainTree` API client method. Let's extend our
`createNotebook` function to create a new chain tree upon successful key
creation, handle any errors that might occur, and let's declare a new variable
to keep track of the new chain tree's id

In file `notebook/index.js`:
```javascript
function createNotebook(creds) {
  let client = connect(creds);
  var keyAddr, chainId;

  client.register()
    .then(function(registerResult){
      return client.generateKey()
    }, function(err) {
      console.log("Error registering wallet.");
      console.log(err);
    }).then(function(generateKeyResult) {
      keyAddr = generateKeyResult.keyAddr;
    }, function(err) {
      console.log("Error generating key.");
      console.log(err);
    }).then(function(createChainResponse) {
      chainId = createChainResponse.chainId;
    }, function(err) {
      console.log("Error creating chain tree.");
      console.log(err);
    });
}
```

### Saving Chain Tree Identifiers
So now we've successfully created both a key and chain tree, and we have the
key's public address and the chain tree's id saved in local variables to prove
it. We still need to keep track of these identifiers between `createNotebook()`
invocations though, so we'll need to save these somewhere more durable than a
local variable. We could use a database for this in a full-fledged production
app, but an external file is enough to serve our purposes.

We can store the key address and the chain id in a JSON object, so let's make a
function that creates the object for us:

In file `notebook/index.js`:
```javascript
...
function identifierObj(key, chain) {
  return {
    keyAddr: key,
    chainId: chain
  };
};
```

Now that we have a way to create our identifier object, we'll need to write that
object to a file. The `fs` module handles file i/o in Node.js, so we'll require
it at the top of `index.js`

In file `notebook/index.js`:
```javascript
const tupelo = require('tupelo-client');
const fs = require('fs');
...
```

Finally, let's pick a relative path to save the identifier file, and write a
function to actually save it

In file `notebook/index.js`:
```javascript
const localIdentifierPath = './.timestamper-identifiers';

function dataFileExists() {
  return fs.existsSync(localIdentifierPath);
}
```

Now that we have all the pieces in place to save the identifiers for the longer
term, let's extend the `createNotebook()` function to persist the identifiers
after successfully creating the chain tree.

In file `notebook/index.js`:
```javascript
function createNotebook(creds) {
  let client = connect(creds);
  var keyAddr, chainId;

  client.register()
    .then(function(registerResult){
      return client.generateKey()
    }, function(err) {
      console.log("Error registering wallet.");
      console.log(err);
    }).then(function(generateKeyResult) {
      keyAddr = generateKeyResult.keyAddr;
    }, function(err) {
      console.log("Error generating key.");
      console.log(err);
    }).then(function(createChainResponse) {
      chainId = createChainResponse.chainId;
      let obj = identifierObj(keyAddr, chainId);
      console.log("Saving registration.");
      return writeIdentifierFile(obj);
    }, function(err) {
      console.log("Error creating chain tree.");
      console.log(err);
    });
}
```
## Adding a Note
Now that we can register a new wallet with the RPC server, create a key, and
create a chain tree, we're ready to start recording notes. Let's build an
`addNote()` function to record our notes. To do that, we'll need to load the
chain tree identifiers from the data file, Read any previously recorded notes,
and append the new note to the chain tree.

### Loading the Identifiers
First, we want to make sure that the user has already created the wallet and
chain tree, and that we already stored the identifiers in the data file. Let's
write a small function to check if the identifier data file exists.

In file `notebook/index.js`:
```javascript
function dataFileExists() {
  return fs.existsSync(localIdentifierPath);
};
```

Next, let's begin writing our `addNote()` function to check if the data file
exists and display an error if not.

In file `notebook/index.js`:
```javascript
function addNote(creds, note) {
  if (!dataFileExists()) {
    console.log("Error: you must register before you can record stamps.");

    return;
  }
};
```

Now we're ready to load the identifiers from the file. Let's write another
function to read and parse the identifiers from the file.

In file `notebook/index.js`:
```javascript
function readIdentifierFile() {
  let raw = fs.readFileSync(localIdentifierPath);
  return JSON.parse(raw);
};
```

Now, let's use that function to save the identifiers in a local variable within
our `addNote()` function

In file `notebook/index.js`:
```javascript
function addNote(creds, note) {
  if (!dataFileExists()) {
    console.log("Error: you must register before you can record stamps.");

    return;
  }

  let identifiers = readIdentifierFile();
};
```

### Reading Previously Recorded Notes
We're ready to use the chain tree identifiers we've just loaded to request the
chain tree from the RPC server. Let's extend our `addNote()` function to connect
to the server, and load the previously stored notes from our chain tree.

First, let's set the path into the chain tree where we'll store (and read) our
notes.

In file `notebook/index.js`:
```javascript
const CHAIN_TREE_NOTE_PATH='notebook/notes'
```

Next, we'll load the data currently at that path using the `resolve()` Tupelo
API method.

In file `notebook/index.js`:
```javascript
function addNote(creds, note) {
  if (!dataFileExists()) {
    console.log("Error: you must register before you can record stamps.");

    return;
  }

  let identifiers = readIdentifierFile();
  let client = connect(creds);

  client.resolve(identifiers.chainId, CHAIN_TREE_NOTE_PATH)
    .then(function(resp) {
    // We'll fill this in later.
  }, function(err) {
      console.log('Error reading notes: ' + err);
  })
};
```

### Adding New Notes
Now we're ready to append our new note to the previously stored notes and write
that data back to the chain tree. But first, we need a way to add a timestamp to
the note we're recording. Let's write a small function to add the current
timestamp to our note.

In file `notebook/index.js`:
```javascript
function addTimestamp(note) {
  let ts = new Date().getTime().toString();
  return ts + '::' + note
};
```

Now let's use this function to add to our `addNote()` function to append the new
note (with timesamp) to the previously stored notes. First, we'll need to make
sure that the data we read from the chain tree is an array and append the new
note. If it isn't an array, then the data is either corrupted or empty, so we
can just discard it for now. We'd add more robust error handling for a
production app though. Lastly, we'll add the newly appended note data back to
the chain tree using the `setData()` Tupelo API method.


In file `notebook/index.js`:
```javascript
function addNote(creds, note) {
  if (!dataFileExists()) {
    console.log("Error: you must register before you can record stamps.");

    return;
  }

  let identifiers = readIdentifierFile();
  let client = connect(creds);

  client.resolve(identifiers.chainId, CHAIN_TREE_NOTE_PATH)
    .then(function(resp) {
    let notes = resp.data,
        noteWithTs = addTimestamp(note);

      if (notes instanceof Array) {
        notes.push(noteWithTs)
      } else {
        notes = [noteWithTs]
      }

      return client.setData(identifiers.chainId,
                            identifiers.keyAddr,
                            CHAIN_TREE_STAMP_PATH,
                            stamps);
  }, function(err) {
      console.log('Error reading notes: ' + err);
  })
};
```


## Viewing the Stored Notes
Since we can save and sign notes to our chain tree, let's print out all the
notes we've recorded so far. We'll write a `printNotes()` function that fetches
the saved notes from the RPC server and prints each one to the console. Just
like our function to save notes, we will also make sure that the user has
already created a wallet and saved some notes.


In file `notebook/index.js`:
```javascript
function printNotes(creds) {
  if (!dataFileExists()) {
    console.log("Error: you must register before you can print stamp tallies.");

    return;
  }

  let identifiers = readIdentifierFile();
  let client = connect(creds);
  let path = CHAIN_TREE_STAMP_PATH;

  client.resolve(identifiers.chainId, CHAIN_TREE_STAMP_PATH)
    .then(function(resp) {
      let notes = resp.dataa

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
};
```

## Adding a Command Line Interface
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
        "tupelo-client": "^0.0.2-alpha1",
        "yargs": "^12.0.2"
    }
}
```

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
yargs.command('register [name passphrase]', 'Register a new notebook chain tree', (yargs) => {
  yargs.positional('name', {
    describe: 'Name of the wallet to save the chain tree.'
  }).positional('passphrase', {
    describe: 'Wallet passphrase.'
  });
}, (argv) => {
  let creds = {
    walletName: argv.name,
    passPhrase: argv.passPhrase
  };

  register(creds);
}).command('add-note [name passphrase]', 'Save a notebook', (yargs) => {
  yargs.positional('name', {
    describe: 'Name of the wallet where  the chain tree is saved.'
  }).positional('passphrase', {
    describe: 'Wallet passphrase.'
  }).describe('n', 'Save a note')
    .alias('n', 'note')
    .demand('n');
}, (argv) => {
  let creds = {
    walletName: argv.name,
    passPhrase: argv.passPhrase
  };

  addNote(creds, argv.n);
}).command('print-notes [name passphrase -n <notes>]', 'Print saved notebooks', (yargs) => {
  yargs.positional('name', {
    describe: 'Name of the wallet where  the chain tree is saved.'
  }).positional('passphrase', {
    describe: 'Wallet passphrase.'
  });
}, (argv) => {
  let creds = {
    walletName: argv.name,
    passPhrase: argv.passPhrase
  };

  printNotes(creds);
}).argv;
```
## Finishing Up
Now we've built a command line notebook that, when invoked with `node`, can
record timestamped notes into a chain tree, print them out later, while the
development notary group validates each note as it's saved.
