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

To achieve concurrent speedups, execution of multiple actions are performed concurrently on forks of the current finalized head, and then [linearized](https://en.wikipedia.org/wiki/Linearizability) and merged \(i.e. the well-used [fork/join model](https://en.wikipedia.org/wiki/Fork%E2%80%93join_model)\). This is less efficient in serial, but has massive improvements in wall-clock time when multiple threads are available \(as is common in IPFS\).

This leverages two facts:

1. The heavy operation is adding the leaves \(files\) to IPFS
2. Concurrent operations are order-independent, and can be rearranged

### Linearization

There are many ways to linearize concurrent updates, all different but equally valid. In the following image, the three sequences on the left are all valid linear orderings for the concurrent partial-order on the right:

![Source: https://noti.st/expede/6IcxBY/tryranny-of-structurelessness\#stLLlcf](../../.gitbook/assets/screen-shot-2021-06-14-at-11.44.22.png)

#### Singleton FIFO Pipeline

We believe that the best tradeoff between raw execution speed and implementation complexity is to respect the order that the user pushes tasks into an optimistic FIFO queue. 

```haskell
data Pending = Pending
  { forkedFrom :: CID
  , update     :: WNFS -> WNFS
  , wip        :: Promise WNFS
  }

data Linearizer = Linearizer
  { pending :: Map CID (Array Pending)
  , openTX  :: WNFS -> WNFS
  , paused  :: Bool
  , local   :: WNFS
  , remote  :: CID
  }
```

New jobs are pushed into a promise at the end of the queue. Work begins immediately, forked from the current finalized `HEAD`. Jobs at the front of the queue `await` the promise.

This uses the [compare-and-set](https://en.wikipedia.org/wiki/Compare-and-swap) strategy. WNFS avoids [the ABA problem](https://en.wikipedia.org/wiki/ABA_problem) thanks to Merkelization: instead of comparing values \(equality\), it checks the root CID \(identity\).

When the front of the queue completes, it checks the finalized root CID of the proposed WNFS's `previous` root CID \(i.e. "has anything changed under me while I was working?"\). If the CIDs match, then this is the next finalized WNFS: place the WNFS in `local`, and \(post-\)compose the transaction's update with the `openTX`.

If they're different:

* Restart the task
* Push the restarted task to the back of the queue
* Merge the first completed promise forked from the current local CID
* Continue as normal

#### Further Optimization

If this proves to be a bottleneck, there are a number of optimizations that can be implemented. One such example is a work-stealing queue which applies deltas as their promises complete. 

This seems an unlikely bottleneck as it only applies when there are many concurrent updates being blocked by a large transaction at the head of the FIFO queue, which is an edge case at best.

## Remote Synchronization

### Push to Remote

When broadcasting the the local version, do the following:

* Pause the transaction queue from merging \(pause merging to `local`\)

On receiving success response:

1. Set the `remote` cache to the latest `local` root CID
2. Set the `openTX` to the identity function
3. Start the transaction queue \(unpause `local`\)

### Pull from Remote

If WNFS is alerted to a new remote HEAD, it does the following:

1. Pause the transaction queue from merging \(pause merging to `local`\)
2. Set `local` and `remote` to the incoming update
3. Re-run `openTX` on top of the new `local` \(blocking, as if at the front of the queue\)
4. Resume merging \(unpause `local`\)

The merge operation is the same as followed during normal head synchronization.

## JS API

What follows is a _sketch_ of a JavaScript API for transactions. Note that these may be automatically retried, so avoiding side effects is strongly suggested \(though not strictly required\). It can be useful to log the number of retries to the console for debugging, or show some feedback to the user, for example.

### atomic

`atomic` is the most flexible, yet lowest-level transactional method. 

It can be used to glue together multiple nested transactions, and provides all other methods are merely special cases of this method.

```typescript
type transaction = {
  id: string;
  iteration: number;
  maxRetries: number;
}

type atomic = <value>(
    fn: {fs: WNFS, tx: transaction} => value,
    txConfig?: {retries?: number}
  ): Promise<{tx: transaction, value: value}>
```

#### Example

```javascript
const photoData = ...

await wnfs.atomic({fs, tx} => {
  fs.photos.insert({
    metadata: {
      takenBy: "expede",
      latitude: 25.025885,
      longitude: -78.035889
    },
    filename: "vacation.png",
    data: photoData
  })
  
  fs.photos.deleteBy({filename: "holiday.gif"})
  
  fs.documents.update({directory, metadata: oldMeta, tx} => {
    directory.contents.find()
    
    return {
      dir: [...directory, newFile],
      metadata: {...oldMeta, lastTouchedBy: "expede"}
    }
  })
})
```

### insert

Insert a completely new file, rather than modifying an existing one.

```typescript
type insert(txConfig?: )
```

### update

```text
wnfs.update(["favourites", "favs.yaml"], data => {
            if data.length > 100 {
              tx.abort({errorMsg: "Data too long"})
            } else {
              const newData = data + " new stuff"
              tx.retryOn(newData.length < 10, {errorMsg: "String too short"})
              return newData.toUpperCase() 
            }
          })
```

### updateBy

### delete

### deleteBy

