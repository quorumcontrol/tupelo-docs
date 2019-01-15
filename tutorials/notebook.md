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

## The Tupelo RPC Server
We will use the Tupelo RPC server to connect our application to the notary
group, so we need to make sure it's installed and running first. Please follow
the [RPC server installation instructions](./rpc_server.md) before continuing
through this tutorial.

## The Tupelo Client API
Now that the RPC server is up and running, we can start building our notebook
application. Before we build anything though, we need to add the Tupelo client
to our application's dependency set. Edit the new `package.json` file in your
favorite editor and add a new dependency set containing only the `tupelo-client`
library:

In file `notebook/package.json`:
```json
{
  "name": "notebook",
  ...
  "dependencies": {
      "tupelo-client": "^0.0.2-alpha1"
  },
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

  // ---- Add the code below ----
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
  var keyAddr; // <--- add a variable for the key address

  client.register()
    .then(function(registerResult){
      return client.generateKey(); // <--- change previous log statement here to
                                   //      a generateKey() call
    }, function(err) {
      console.log("Error registering wallet.");
      console.log(err);
    }).then(function(generateKeyResult) {
      keyAddr = generateKeyResult.keyAddr; // <--- save the key address here
    }, function(err) {
      console.log("Error generating key.");
      console.log(err); // <--- log any generateKey() errors here
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
  var keyAddr, chainId; // <--- add a new variable for the chain tree id

  client.register()
    .then(function(registerResult){
      return client.generateKey();
    }, function(err) {
      console.log("Error registering wallet.");
      console.log(err);
    }).then(function(generateKeyResult) {
      keyAddr = generateKeyResult.keyAddr;
      return client.createChainTree(keyAddr); // <--- add createChainTree()
                                              //      request
    }, function(err) {
      console.log("Error generating key.");
      console.log(err);
    }).then(function(createChainResponse) {
      chainId = createChainResponse.chainId; // <--- save the chain id here
    }, function(err) {
      console.log("Error creating chain tree.");
      console.log(err); // <--- log and createChainTree() errors here
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
}
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
const LOCAL_ID_PATH = './.timestamper-identifiers';
...
function writeIdentifierFile(configObj) {
  let data = JSON.stringify(configObj);
  fs.writeFileSync(LOCAL_ID_PATH, data);
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
      console.log("Saving registration.");       // <--- log a "save" message
      let obj = identifierObj(keyAddr, chainId); // <--- build identifier object
      return writeIdentifierFile(obj);           // <--- write identifier file
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
function idFileExists() {
  return fs.existsSync(LOCAL_ID_PATH);
}
```

Next, let's begin writing our `addNote()` function to check if the data file
exists and display an error if not.

In file `notebook/index.js`:
```javascript
function addNote(creds, note) {
  if (!idFileExists()) {
    console.log("Error: you must register before you can record notes.");

    return;
  }
}
```

Now we're ready to load the identifiers from the file. Let's write another
function to read and parse the identifiers from the file.

In file `notebook/index.js`:
```javascript
function readIdentifierFile() {
  let raw = fs.readFileSync(LOCAL_ID_PATH);
  return JSON.parse(raw);
}
```

Now, let's use that function to save the identifiers in a local variable within
our `addNote()` function

In file `notebook/index.js`:
```javascript
function addNote(creds, note) {
  if (!idFileExists()) {
    console.log("Error: you must register before you can record notes.");

    return;
  }

  let identifiers = readIdentifierFile(); // <--- read the identifiers here
}
```

### Reading Previously Recorded Notes
We're ready to use the chain tree identifiers we've just loaded to request the
chain tree from the RPC server. Let's extend our `addNote()` function to connect
to the server, and load the previously stored notes from our chain tree.

First, let's set the path into the chain tree where we'll store (and read) our
notes.

In file `notebook/index.js`:
```javascript
const CHAIN_TREE_NOTE_PATH='notebook/notes';
```

Next, we'll load the data currently at that path using the `resolve()` Tupelo
API method.

In file `notebook/index.js`:
```javascript
function addNote(creds, note) {
  if (!idFileExists()) {
    console.log("Error: you must register before you can record notes.");

    return;
  }

  let client = connect(creds), // <--- connect the client
      identifiers = readIdentifierFile();

  client.resolve(identifiers.chainId, CHAIN_TREE_NOTE_PATH)
    .then(function(resp) {
    // We'll fill this in later.
  }, function(err) {
      console.log('Error reading notes: ' + err);
  });
}
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
  return ts + '::' + note;
}
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
  if (!idFileExists()) {
    console.log("Error: you must register before you can record notes.");

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
                            CHAIN_TREE_NOTE_PATH,
                            notes);
  }, function(err) {
      console.log('Error reading notes: ' + err);
  })
}
```

## Viewing the Stored Notes
Since we can save and sign notes to our chain tree, let's print out all the
notes we've recorded so far. We'll write a `showNotes()` function that fetches
the saved notes from the RPC server and prints each one to the console. Just
like our function to save notes, we will also make sure that the user has
already created a wallet and saved some notes.


In file `notebook/index.js`:
```javascript
function showNotes(creds) {
  if (!idFileExists()) {
    console.log("Error: you must register before you can print notes.");

    return;
  }

  let identifiers = readIdentifierFile();
  let client = connect(creds);

  client.resolve(identifiers.chainId, CHAIN_TREE_NOTE_PATH)
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
      console.log('Error fetching notes: '  + err);
    });
}
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

## Finishing Up
Now we've built a command line notebook that, when invoked with `node`, can
record timestamped notes into a chain tree, print them out later, while the
development notary group validates each note as we save them. Be sure to take a
look at the final [`package.json` file](/tutorials/notebook/package_json) and
the final [`index.js` file](/tutorials/notebook/index_js).
