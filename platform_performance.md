---
layout: default
title: Platform Performance
has_children: false
nav_order: 5
---

# Latest Tupelo Performance Data

The engineering team focusing on Tupelo take performance extremely seriously.  The best way we can live that is by sharing, as transparently as possible, what has been measured, how the Test-Net has performed and what tests we are planning.

Reliable performance information on most blockchain and DLT projects is extremely hard to come by.  Incredibly optimistic scenarios are built, whole section of the infrastructure are "skipped" or "estimated", and unsubstantiated claims are made.  On the flip side currently running networks have real numbers but they are nearly all underwhelming.

The best we can do is be as transparent with all of the performance data we have available.  Sometimes this will be good, others attempted improvements will be a step in the wrong direction.  Below you will find a latest and we welcome questions and suggestions in our Telegram room if you have ideas on how we can improve.

***

## December 18th - Latest update
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
## November 29th - Turkey Day Surprise
TLDR: Updated test methodologies to improve the simulation of a variety of heavier-weight transactions made things a little worse but more realistic.

| Signers | Throughput  | Finality (75%)  |
| ------- |:-----------:|:---------:|
| 20      | 91 tx/sec   | 1.02 s    |
| 50      | 84 tx/sec   | 2.15 s    |
| 100     | 18 tx/sec   | 2.95 s    |

[For more details about this test](/platform_performance)

***
