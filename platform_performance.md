---
layout: default
title: Platform Performance
has_children: false
nav_order: 5
---

# Latest Tupelo Performance Data

The engineering team focusing on Tupelo take performance extremely seriously.  The best way we can live that is by transparently sharing what has been measured, how the Test-Net has performed and what tests we are planning.

Our current tests are being done with a 21 and 99 node blah blah blah
Signers are spread out across the world and numbers include blah blah

We will continue to share these exciting numbers as they become available.

***

## January 6th - Latest update
TLDR: Key changes were made to optimize the inverse bloom filters and it showed significant improvements across the board.

| Signers | Throughput  | Finality (75%)  |
| ------- |:-----------:|:---------:|
| 20      | 85 tx/sec   | 1.22 s    |
| 50      | 74 tx/sec   | 2.35 s    |
| 100     | 25 tx/sec   | 2.85 s    |

[For more details about this test](/platform_performance)

***
## December 11th Performance Test
TLDR: Integrated a high-performance WASM BLS signatures library which resulted in some improvements in both throughput and time to finality.

| Signers | Throughput  | Finality (75%)  |
| ------- |:-----------:|:---------:|
| 20      | 91 tx/sec   | 1.02 s    |
| 50      | 84 tx/sec   | 2.15 s    |
| 100     | 18 tx/sec   | 2.95 s    |

[For more details about this test](/platform_performance)

***
