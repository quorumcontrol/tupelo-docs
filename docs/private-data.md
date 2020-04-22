---
layout: default
title: Private Data on Tupelo
parent: Platform Documentation
nav_order: 4
has_mermaid: true
---
# Private Data on Tupelo

Building on the Tupelo public network does not mean data needs to be open to the public.  The Tupelo network is permissionless (unless run in a consortium mode), but the underlying data can be kept private. A group can keep data private to its members while gaining the increased security and interoperability of a permissionless DLT.

There are multiple different ways to achieve data privacy on Tupelo. Tupelo separates storage and trust which opens up several unique options. 


<img style="float: right; width: 395px; height: 516px; " src="../assets/images/TupeloPrivateData.png">

One highly integrated data privacy option is to use the "on ChainTree" access control feature of Tupelo.  An access control list is incorporated into each ChainTree by its creator/owner. When (1) data is accessed via the Tupelo storage service, (2) the ACL is checked before returning any data.  Identified users, as recognized by their public key, are granted access if and only if they are on the ACL list for that portion of the ChainTree.  

Only the current owner of the ChainTree may alter the access control list.  Tupelo enables this option through a customized version of bitswap which does the ACL check whenever blocks are accessed.

Maintaining a consortium ChainTree that includes the public keys of all members enables them to share data privately and securely. During the creation of new objects those keys are written to the appropriate ACL list.   

Mixes of data including fully public, private to a consortium and private to the owner, are all possible within a single tree.  Outside of the fully public option, signers in the Tupelo network would not have access to the private data but would be signing the updates (hashes and changes to those hashes) to provide trust.  

An alternative to using Tupelos integrated ACL feature is for consortiums to manage their own data encryption and storage.  Tupelo can be used to sign ChainTree updates (CID changes), retaining the integrity and immutability of the underlying data, without any public exposure.  Any party with rights to the underlying files can then access the data, in combination with the signed CIDs provided by Tupelo, to confirm both the current state and the history of that particular object.

The privacy option is uniquely available on Tupelo because of the unique structure of ChainTrees.  Tupelo enables sensitive private data shared between parties without the expense, overhead, and maintenance of a private blockchain.



