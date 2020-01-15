---
layout: default
title: Hello Tupelo
parent: Tutorials
has_children: true
nav_order: 1
---

# Hello Tupelo
{: .fs-9 }

Hello World makes a good first step to explore a new language or platform.

It is worth noting that through these simple steps the data is being
submitted (via a Chain Tree) and signed by a Notary group.  

To the developer it feels like storing it to any datastore such as a
document database.  After that submit however, the information is being reviewed,
confirmed valid and made immutable by the network. Tupelo makes DLTs easy.  

# Hello from Shell

First you will need to [download Tupelo](https://github.com/quorumcontrol/tupelo/releases).



Unzip tupelo-v0.0.0.zip (where 0.0.0 is the actual version number you downloaded)
and from a terminal rename whichever version you need based on which OS you are running.  
Then, chmod it to make it executable.

```
mv tupelo-v0.5.11-darwin-amd64 tupelo
chmod +x tupelo
```

Next follow these steps from a command shell (stepping past OS publisher checks if they appear)
to quickly write some data to a chain tree and read it back.

```
$ ./tupelo shell -w billfold
>>> create-wallet
>>> start-session
>>> create-key
>>> create-chain <key address>
>>> set-data <chain tree id> <key address> my/data/path "hello tupelo"
>>> resolve-data <chain tree id> my/data/path
>>> stop-session
```

**Congratulations!**  
You have successfully written to a ChainTree and your request was verified by the Tupelo network.

## Commentary
Read below if you'd like an explanation of what each of the above steps does.

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
>>> resolve-data <chain tree id> my/data/path
data: hello tupelo
remaining path: []
```

The next step in getting to know the Tupelo DLT is to try our [notebook tutorial](/tutorials/notebook).
