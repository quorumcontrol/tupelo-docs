---
layout: default
title: Tutorials
has_children: true
nav_order: 2
---

# Tupelo Tutorials

In support of helping developers get started we have created a series of tutorials.
We start with the most simple and get progressively more sophisticated.

***

## Hello Decentralized World

The obligatory first tutorial is the "Hello World" app.
For this project it is a little more appropriate than most, as we will be
creating a message and getting it signed by the Tupelo Network. We are actually
saying "Hello" to this new decentralized "World."

This should only take a few minutes and we will have a Chain-Tree that contains
immutable proof that we said Hello to the Tupelo World on this day.

[Hello Decentralized World Tutorial](tutorials/hello_tupelo){: .btn .btn-green .fs-5 .mb-4 .mb-md-0 .mr-2 }

***

## Notebook App

One of the use cases for blockchain is in recording information that is
immutable.  This walkthrough takes you through the building of a simple
Javascript app which allows the user to "stamp" notebook entries.
That means they will be signed by the Tupelo network and that what
was written and when can be proven at any time in the future.

[Notebook App Tutorial](tutorials/notebook){: .btn .btn-green .fs-5 .mb-4 .mb-md-0 .mr-2 }

***

## RPC Server

The Node.js client cannot directly manage chain trees or connect to the notary
group, so node applications must instead proxy through an RPC server to work
with Tupelo.  This brief tutorial explains how to get that setup.

[RPC Server Tutorial](tutorials/rpc_server){: .btn .btn-green .fs-5 .mb-4 .mb-md-0 .mr-2 }

***

{% include feedback.html %}
