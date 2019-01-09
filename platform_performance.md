---
layout: default
title: Platform Performance
has_children: false
nav_order: 5
---

# Latest Tupelo Performance Data

We take performance seriously.  

The best way we can live that is by transparently sharing what has been
measured, how the Test-Net has performed and what changes are driving
performance improvements.

Our current tests are being done with 21 and 100 signing nodes.
We have been expanding the number of AWS regions to better model latency as our
testing processes have been getting more robust.  The number of regions and
instance type are noted with the results below.

We will continue to share these exciting numbers as they become available.

***

## January 6th - Latest update
Refactoring workflow to use actor model [ProtoActor](http://proto.actor).
Adding additional in-memory state.

| Signers | Throughput  | Finality (mean)  | Finality (P95)  |
| ------- |:-----------:|:---------:|:---------:|
| 21      | 200 tx/sec   | 1814 ms  | 2367 ms |
| 100     | 200 tx/sec   | 4612 ms  | 6104 ms |

AWS Regions: 8  
AWS Instance Types: c5.xlarge (4cpu 8gb ram)  

***

## December 11th Performance Test

Moved ingress of messages on a signer to an in memory queue (instead of disk).
Parallelized signature checking.

| Signers | Throughput  | Finality (mean)  | Finality (P95)  |
| ------- |:-----------:|:---------:|:---------:|
| 21      | 50 tx/sec   | 1218 ms  | 1791 ms |
| 100     | 25 tx/sec   | 3662 ms  | 4525 ms |

AWS Regions: 8  
AWS Instance Types: c5.xlarge (4cpu 8gb ram)  

***

## December 4th Performance Test

Shifted to IBF (Invertible Bloom Filter) for transaction/state syncing
between signer nodes.

| Signers | Throughput  | Finality (mean)  | Finality (P95)  |
| ------- |:-----------:|:---------:|:---------:|
| 21      | 50 tx/sec   | 1147 ms  | 2320 ms |
| 100     | 25 tx/sec   | 12143 ms  | 19434 ms |

AWS Regions: 3  
AWS Instance Types: c5.xlarge (4cpu 8gb ram)  
