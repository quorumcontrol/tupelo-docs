---
layout: default
title: IoT Sensor Tutorial
parent: Tutorials
nav_order: 3
---

# IoT Sensor Tutorial
{: .fs-9 }

# Getting Started with Tupelo

This tutorial assumes that you have already downloaded and installed the Tupelo.JS client and have setup the appropriate RPC.
If you have not you [should do that first.](api_reference/js_api)

First lets set the stage.  A small inexpensive temperature measurement device is being included with a shipment of mussels to ensure they are kept at precisely the right temperature as they make their way to the finest NYC restaurants.  First they are caught and put on ice in a large batch and our IoT device is switched on and put on top.  Once they get back to port they are transferred to the distributor.  The distributor then takes them in a truck to Chez Fancy Pants.  All the while our device is checking the temperature and recording it in a DLT as proof of both time to market and that proper care was taken along the way in handling them.

Now what does that look like in code.  Unlike other DLT or blockchain solutions we don't need complex smart contracts to make this work.  We only need to create the device, assign a batch identifier and transfer that as it passes through the supply chain to market.

Then we will create a single simple JS file.

Then we will call it from the command line and say hello to the decentralized world
