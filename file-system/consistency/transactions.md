# Transactions

WNFS is transactional, lock-free, and concurrent. Updates are made atomically, and bundling multiple actions together is allowed. This is achieved by maintaining a local fork of the state, and only persisting it to the network \(an irreversable comittment\) when it is certain of its state. 

Bundling multiple states has many advantages, including lower synchronization overhead, 

Atomic updates can be associativity merged locally before being shared with the network. These are stored as closures, and run on the \*\*\*. _Suspended_ \_\_\_\_\_\_\_. They can be re-run before being settled.

This is simple function composition, and forms a monoid on transactions:

$$
(Transaction, ∘, λx.x)
$$

  


## Mechanics

### Out of Order Execution

### Linearization

### Compare-and-Swap

