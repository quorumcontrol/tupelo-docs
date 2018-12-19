---
layout: default
title: Whitepaper v0.10
parent: Platform Documentation
nav_order: 3

---

# Tupelo Whitepaper

A better way forward for digital asset ownership: <br>
Chain Trees & Notary Groups: Tupelo Working Draft - October 2018 - v0.10

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

Abstract

Digital asset ownership systems can achieve faster processing time, more decentralization, and less overhead by adopting a client-lead consensus algorithm and removing global transaction ordering. Introducing an individual append-only log (blockchain) combined with a notarized merkle-DAG (directed acyclic graph) per actor allows for a variety of protocols to be built on top of a byzantine fault tolerant global layer of trust while scaling to hundreds of thousands of transactions per second. We call this combination of blockchain and merkle-DAG a Chain Tree. The combination of a Chain Tree and a public Notary Group is a system we are calling Tupelo.

## Background
The current state of the art in cryptocurrency systems is referred to as “distributed ledger technology (DLT).” Most DLT systems model asset ownership/transfer as quantities sent between accounts within atomic transactions. If Alice wants to send Bob 5 tokens, she creates a transaction on the ledger which debits her account 5 tokens and increases Bob’s account 5 tokens. In order to prevent Alice from double spending, Bob has to wait until the ledger has come to a consensus that Alice’s transaction is valid. The distributed ledger needs to keep a global ordered transaction log to make sure that Alice has the 5 tokens to spend. Global ordering is one of the reasons many distributed ledgers can only process tens of transactions per second globally.

Additionally, these systems are not suited for real-world asset ownership. Distributed ledgers were designed to model currency and similar concepts such as stocks and bonds, but there are few systems where transferring ownership of items like real estate or cars (or door locks, routers, etc) is ideal. Developers have used existing systems as timestamp generators, but the systems themselves aren’t designed for individual objects.

Tupelo builds upon the work of others including:
Cosi/Cothority
BLS Multisignature
W3C DID spec
Secure Scuttlebutt
Nano
Holochain
Gosig
Casper FFG
IPLD

## Overview
We model asset ownership with a new data structure called a Chain Tree.  Every asset and actor in the system has their own Chain Tree. A Chain Tree is the combination of a merklized DAG and an individual ordered log of transactions. This structure can be thought of as a single-branch git for data.

A group of Signers (as part of a Notary Group) keeps track of the current state of every Chain Tree in order to prevent forking. The Notary Group does not need to keep the entire history nor the entire state of every Chain Tree, but only the latest tip (a hash of the current root node).

We are able to do away with all Signer-to-Signer communication for transaction signing using a new client-driven consensus protocol.

Putting these concepts together allows for a high level of parallelization and a dramatic decrease in message volume, while maintaining the same, high, level of security as traditional DLT.

## Chain Tree ##

A Chain Tree is similar in concept to Git, but with known (and, later, user-definable) transactions on data instead of simple text manipulation. A Chain Tree is a state-machine where the input and resulting state is a content-addressable merkle-DAG. Playing ordered transactions on an existing state produces a new state. We use the IPLD format to model Chain Trees.

This whitepaper won’t go into the details of IPLD, but, at a high level, IPLD specifies how to link data structures using a content addressable system (hashing). Conceptually similar to JSON, Tupelo uses CBOR (Compact Binary Object Representation) to model the data inside a ChainTree. CBOR specifies a canonical way to create a binary given key/value pairs. An object can link to another object by specifying a CID as a value within one if its key/value pairs. A CID represents a hash of the object linked TO. In this way, a single tip (hash of the root object) can be used to verify that all children of the object have not been tampered with.

To illustrate this data structure, we will take a look at a couple of transactions, starting very simply, then moving to slightly more complex.


Figure 1 - A simple transition

In Figure 1, We can see that there was an empty tree, we played the SET_DATA transaction on the tree and ended up with a new tip for the tree and a new entry in our chain. This is the simplest possible scenario, where the “tree” is actually just a single node and we are just setting data on the root node. The result is a data structure of the combination of a 1 node chain and a 1 node tree.


Figure 2 - Two new transaction types

In Figure 2, there are two separate transactions taking place on the Chain Tree created in Figure 1. The State Machine acting on this Chain Tree knows how to handle an ADD_AUTHENTICATION and it does so by adding a link to the Authentications on the root node to a list of authentications and appending this entry to that list (in this case: an empty set). Next, we play a RM_AUTHENTICATION transaction on the Chain Tree which removes the node created by the ADD_AUTHENTICATION. Playing the chain on the right will always result in the same tree created on the left. The tip of the Chain Tree is created by hashing the root node. This allows for a content-addressable tree of unlimited size to be signed by a single hash value.

## Actors

Every actor in the system (both identities and assets) may have one or many Chain Tree(s). Actors create a new, global, Chain Tree by creating a new public/private secp256k1 key-pair. The Chain Tree’s ID is a DID derived from the ethereum-style address of the public key (created the same way that Ethereum creates addresses based on public keys). An example Chain Tree DID is: did:tupelo:0xDD4f79D433FC132dc22e662F25B5D46fADFfa16. Before any transactions have been played on the Chain Tree, the Chain Tree is in the genesis state. The owner of a genesis Chain Tree is the holder of the private key that corresponds to the address in the Chain Id.

Ownership of a Chain Tree may be changed by adding public keys to a system-defined path in the tree. In the case of this system the path /_tupelo/authorizations should contain a set of links to public keys. If /_tupelo/authorizations is defined on the Chain Tree then this ownership overrides the keypair used to create the Chain Tree.

Only the owner(s) of a Chain Tree may create new transactions on a Chain Tree. The tip of a Chain Tree refers to the hash of the root node of the current state of a Chain Tree (a Chain Tree with all transactions applied).

Transactions are arbitrary mutations of state. The client and Notary Group must agree on the process of mutating state through named transactions. The initial release will include the following transactions:
SET_DATA
SET_OWNERSHIP
ESTABLISH_COIN
MINT_COIN
SEND_COIN
RECEIVE_COIN

Later, we will discuss how user-defined transactions can be used to expand the system in a manner similar to smart-contracts on other DLT systems.

Payments Through Cryptocurrency
Initial implementations include a protocol to transfer quantities of a token. Conceptually this is the same as any other cryptocurrency token such as Ethereum or Bitcoin. Chain Trees and Notary Groups accomplish double-spend protection on top of the existing layer of trust described above and do not need a global ledger. This system allows for unlimited currencies on the same notarized system without the need for smart contracts.

Currency exchange introduces 4 transaction types: ESTABLISH_COIN, MINT_COIN, SEND_COIN, RECEIVE_COIN. The Chain Tree structure is modeled as in Figure 3. This layout allows for efficient double-spend checking by sending in all the receives. However, a future improvement to the protocol will use balance check pointing and a cryptographic accumulator (or zk-snark) to prove non-existence of the receive.
