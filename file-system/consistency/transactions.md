# Transactions

WNFS is transactional, lock-free, and concurrent. Updates are made atomically, and bundling multiple actions together is both allowed and encouraged. This is achieved by maintaining a local fork of the state, and only persisting it to the network \(an irreversible comittment\) when it is certain of its state. Bundling multiple states has many advantages, including lower synchronization overhead.

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
  /////////////////
 // Rough Types //
/////////////////

class Transaction = {
  id: string;
  
  iteration: number;
  maxRetries: number;
  
  abort: (msg: string) => void;
  failOn: (predicate: any => boolean, msg: string) => void;
}

type txConfig = {retries: number} | {rootTx: transaction}

type atomic = <value>(
    fn: {fs: WNFS, tx: transaction} => value,
    txConfig?: txConfig
  ): Promise<{tx: transaction, value: value}>

  ////////////////////
 // Example Sketch //
////////////////////

const me = "expede"
const photoData = ...
const nestedTx = async () => wnfs.atomic({fs, tx} => {
  // more transactional actions
})

await wnfs.atomic({fs, tx} => {
  fs.create(["photos", "vacation.png"], {
    metadata: {
      takenBy: me,
      latitude: 25.025885,
      longitude: -78.035889
    },
    data: photoData
  })
  
  fs.rm(["public", "photos", "holiday.gif"])
  
  // Nested transaction
  await nestedTx().bind(this) // or something
  
  fs.modify(["public", "documents"], {directory, metadata: oldMeta, tx} => {
    directory.contents.find()

    return {
      dir: [...directory, newFile],
      metadata: {...oldMeta, lastTouchedBy: me}
    }
  })
}).maxRetries(0)
```

### create

```typescript
// The path will be the 
type create(path: path, data: Uint8Array, meta?: metadata): Promise<>

// Example

wnfs.create(["private", "Documents"], data,)
```

### modify

```typescript
wnfs.modify(["favourites", "favs.yaml"], data => {
  if (data.length > 100) throw new Error("Data too long")
  const newData = data + " new stuff"
  
  if (newData.length < 10) throw new Error("String too short")
  return newData.toUpperCase() 
})
```

### remove

```typescript
wnfs.remove(["favourites", "favs.yaml"])
```



