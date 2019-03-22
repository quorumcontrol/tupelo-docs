---
layout: default
title: Hello from Shell
parent: Hello Tupelo
grand_parent: Tutorials
nav_order: 1
---

# Hello from Shell

## Quickstart
Follow these steps from a command shell to quickly write some data to a chain tree and read it back.

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
