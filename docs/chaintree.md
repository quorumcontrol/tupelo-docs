# What's a ChainTree?

## Overview

A ChainTree is a novel data structure used in Tupelo to represent objects (digital and physical).

As the name implies, a ChainTree is a combination of a blockchain and a tree for the current state. That means that at any point, a ChainTree represents both its current state and the entire history of changes that took it to its current state.

```mermaid
graph TD
Root --> Tree
Root --> Chain
subgraph Tree
  Tree --> Arb["Arbitrary Tree"]
  Arb --> Anything
  Arb --> uw["user wants"]
  Tree --> _tupelo
  _tupelo --> Coins
  _tupelo --> Ownership
  Ownership --> PublicKeys
end
subgraph Chain
Chain -- Latest --> b3["Block 3"]
Chain -- Genesis --> gen
b2["Block 2"] --> b3 
gen["Genesis"] --> b2
end
```

A ChainTree is a [content addressable](https://en.wikipedia.org/wiki/Content-addressable_storage) data structure, meaning that the entire data structure can be represented by a single hash of the root. 

ChainTrees use [IPLD](https://ipld.io/) for internal node linking. 

From [Tupelo's whitepaper](https://docs.quorumcontrol.com/docs/whitepaper.html):
> [...] at a high level, IPLD specifies how to link data structures using a content addressable system (hashing). Conceptually similar to JSON, Tupelo uses CBOR (Compact Binary Object Representation) to model the data inside a ChainTree. CBOR specifies a canonical way to create a binary given key/value pairs. An object can link to another object by specifying a CID as a value within one if its key/value pairs. A CID represents a hash of the object linked TO. In this way, a single tip (hash of the root object) can be used to verify that all children of the object have not been tampered with.

This data structure is the super power behind Tupelo. It allows tupelo to reach consensus on a simple hash of the root object and only store the current root of individual objects. The relevant history and state can be passed back in during transactions and Tupelo can know this state is correct because it has the current hash of the root.

To reiterate, the ChainTree root links (using a [CID](https://github.com/multiformats/cid), from IPLD) to a tree (which is the current state) and a chain, which is a blockchain of the transactions used to get the tree from genesis (blank) state to the current state. This allows you to store arbitrary data in your ChainTree and use the IPLD standard to navigate through the nodes of the Tree.

Modifying a ChainTree involves playing transactions against the current state to get to a new state. Those transactions happen in blocks and become part of the chain section of the ChainTree.

For instance, given the following ChainTree:
```mermaid
graph TD
Root --> Tree
Root --> Chain
end
```

I could run a "set data" transaction on the chaintree:

```json
{
	"type": "SET_DATA",
	"payload": {
	  "path": "/my/subtree/foo",
	  "value": "bar"
    }
}
```

That would result in the following chain tree:


```mermaid
graph TD
Root --> tree
Root --> Chain
subgraph Tree
  tree["<strong>tree</strong><br/><code>{
  my: myCid}</code>"] --> subtree["<strong>subtree</strong><br/><code>{
  foo: 'bar'}</code>"]
end
subgraph Chain
Chain -- Latest --> b1["block"]
Chain -- Genesis --> b1
end
```

And block would look like this:
```json
{
	"headers": {
		"signatures": {
			"owner": "asignature",
		},
	},
	"block": {
		"previousTip": "",
		"transactions": [{
		  "type": "SET_DATA",
	      "payload": {
			"path": "/my/subtree/foo",
		    "value": "bar"
          }  
		}]
	}
}
```

Using the set data transactions, you can set data to any part of the tree and the history of those changes is preserved in the data structure.



