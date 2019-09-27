---
layout: default
title: Platform Performance
has_children: false
nav_order: 6
---

# Latest Tupelo Performance Data

We take performance seriously.  

The best way we can live that is by transparently sharing what we have
measured, how the test network has performed and what changes are driving
performance improvements.

We are currently testing on 21 and 100 signing node clusters.
We have been expanding the number of AWS regions to better model latency.
The number of regions and instance type are noted with the results below.

We will continue to share these numbers as they become available.

***

## September 27th - Latest update

Many updates have been made since the last performance update but most recently
Tupelo has been updated to go 1.13.

| Signers | Throughput  | Finality (mean)  | Finality (P95)  |
| ------- |:-----------:|:---------:|:---------:|
| 21      | 200 tx/sec   | 144 ms  | 188 ms |
| 21      | 250 tx/sec   | 311 ms  | 496 ms |

AWS Regions: 8  
AWS Instance Types: c5.xlarge (4cpu 8gb ram)  
Tupelo Version: 0.5.0

***

## June 6th
Improvements largely driven by the change of core Tupelo types to protocol
buffer messages and the use of those across all platform-specific development
kits.

| Signers | Throughput  | Finality (mean)  | Finality (P95)  |
| ------- |:-----------:|:---------:|:---------:|
| 21      | 200 tx/sec   | 160 ms  | 246 ms |
| 100     | 200 tx/sec   | 297 ms  | 469 ms |

AWS Regions: 8  
AWS Instance Types: c5.xlarge (4cpu 8gb ram)  
Tupelo Version: 0.4.0

***

## May 17th
Substantial performance improvements upgrading libp2p pubsub, fixing a problem
with rapid subscriptions, and improving our benchmarking methodology.

| Signers | Throughput  | Finality (mean)  | Finality (P95)  |
| ------- |:-----------:|:---------:|:---------:|
| 21      | 200 tx/sec   | 197 ms  | 290 ms |
| 100     | 200 tx/sec   | 423 ms  | 664 ms |

AWS Regions: 8  
AWS Instance Types: c5.xlarge (4cpu 8gb ram)  
Tupelo Version: 0.2.3

***

## April 19th
Substantial performance improvements through consensus streamlining including
optimizing subscriptions and improved bootstrapping.

| Signers | Throughput  | Finality (mean)  | Finality (P95)  |
| ------- |:-----------:|:---------:|:---------:|
| 21      | 200 tx/sec   | 839 ms  | 1381 ms |
| 100     | 200 tx/sec   | 831 ms  | 1739 ms |

AWS Regions: 8  
AWS Instance Types: c5.xlarge (4cpu 8gb ram)  

***

## January 6th
The workflow was refactored to use actor model [ProtoActor](http://proto.actor).
More of the state was moved to be held in-memory.

| Signers | Throughput  | Finality (mean)  | Finality (P95)  |
| ------- |:-----------:|:---------:|:---------:|
| 21      | 200 tx/sec   | 1814 ms  | 2367 ms |
| 100     | 200 tx/sec   | 4612 ms  | 6104 ms |

AWS Regions: 8  
AWS Instance Types: c5.xlarge (4cpu 8gb ram)  

***

## December 11th Performance Test

Message ingress was moved to an in memory queue (from disk).
Signature checking was parallelized.

| Signers | Throughput  | Finality (mean)  | Finality (P95)  |
| ------- |:-----------:|:---------:|:---------:|
| 21      | 50 tx/sec   | 1218 ms  | 1791 ms |
| 100     | 25 tx/sec   | 3662 ms  | 4525 ms |

AWS Regions: 8  
AWS Instance Types: c5.xlarge (4cpu 8gb ram)  

***

## December 4th Performance Test

Transaction/state syncing between nodes was changed to an IBF
(Invertible Bloom Filter).

| Signers | Throughput  | Finality (mean)  | Finality (P95)  |
| ------- |:-----------:|:---------:|:---------:|
| 21      | 50 tx/sec   | 1147 ms  | 2320 ms |
| 100     | 25 tx/sec   | 12143 ms  | 19434 ms |

AWS Regions: 3  
AWS Instance Types: c5.xlarge (4cpu 8gb ram)  
