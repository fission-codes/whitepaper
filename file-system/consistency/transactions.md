# Transactions

WNFS is transactional, lock-free, and concurrent. Updates are made atomically, and bundling multiple actions together is allowed. This is achieved by maintaining a local fork of the state, and only persisting it to the network \(an irreversable comittment\) when it is certain of its state. 

Bundling multiple states has many advantages, including lower synchronization overhead, 

Atomic updates can be associativity merged locally before being shared with the network. These are stored as closures, and run on the \*\*\*. _Suspended_ \_\_\_\_\_\_\_. They can be re-run before being settled.

This is simple function composition, and forms a monoid on transactions:

$$
(Transaction, ∘, λx.x)
$$

  
While suspended, the local copy can continue to accept updates without creating a confluent branch. This is desirable since it avoids runtime linearlization.

## Mechanics

### Out-of-Order Execution

To achieve concurrent speedups, execution of multiple actions are performed concurretly on forks of the current finalized head, and then [linearized](https://en.wikipedia.org/wiki/Linearizability) and merged \(i.e. the well-used [fork/join model](https://en.wikipedia.org/wiki/Fork%E2%80%93join_model)\). This is less efficient in serial, but has massive improvements in wall-clock time when multiple threads are available \(as is common in IPFS\).

This leverages two facts:

1. The heavy operation is adding the leaves \(files\) to IPFS
2. Concurrent operations are order-independent, and can be rearranged

### Linearization

There are many ways to linearize concurrent updates, all different but equally valid. In the following image, the three sequences on the left are all valid linear orderings for the concurrent partial-order on the right:

![Source: https://noti.st/expede/6IcxBY/tryranny-of-structurelessness\#stLLlcf](../../.gitbook/assets/screen-shot-2021-06-14-at-11.44.22.png)

We believe that the best tradeoff between raw execution speed and implementation complexity is to respect the order that the user pushes tasks into a FIFO queue \(see below\). If this proves to be a bottleneck, it can be swapped for a work-stealing queue, that applies deltas as their promises complete. This seems an unlikely bottleneck as it only applies when there are many concurrent updates being blocked by a large transaction at the head of the FIFO queue, which is an edge case at best.

#### Singleton FIFO Queue



#### Compare-and-Swap



WNFS avoids [the ABA problem](https://en.wikipedia.org/wiki/ABA_problem) thanks to Merkelization. Instead of comparing values \(equality\), we check the root CID \(identity\).

### Head Update Rebase

The merge operation is the same as in the 

![](https://memegenerator.net/img/instances/50284181/this-never-happened.jpg)

