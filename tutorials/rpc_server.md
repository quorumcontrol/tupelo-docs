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

First you will need to [download Tupelo](https://github.com/quorumcontrol/tupelo/releases).
Unzip tupelo-v0.5.11.zip and rename whichever version you need based on which OS you running.
Next, chmod it to make it executable.

```bash
mv tupelo-v0.5.11-darwin-amd64 tupelo
chmod +x tupelo
```

Once you have selected and renamed the appropriate binary for your platform save it within
your command `PATH` variable. If you do not wish to save the binary to a
directory in your `PATH`, you can still execute it with the fully qualified
or relative path to your chosen location for the binary.

You can run the network locally or connect to our TestNet.

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

## Connecting the RPC Server to a TestNet

You have the option to connect to our TestNet.
This TestNet is shared among our community of developers and currently has
21 signing nodes running.

To connect to the TestNet, you'll need to set the TUPELO_BOOTSTRAP_NODES env
variable - you'll find the current value inside bootstrap.env in
[testnet-keys.zip](https://qc-tupelo-downloads.s3.eu-central-1.amazonaws.com/testnet-keys.zip).

With that environment variable set and the zipfile expanded, you can now run
tupelo with:
```bash
./tupelo rpc-server -k testnet-keys/
```

That sample command assumes the Tupelo binary is next to the extracted
testnet-keys.zip, but -k can take a full path if you put the file somewhere
else.

Note there are other TestNets available if the one provided does not meet your
needs.  Please reach out via the feedback form below and let us know how we can
help or what issues you might be having.

***

{% include feedback.html %}
