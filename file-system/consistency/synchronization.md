# Synchronization

The Web Native File System is a functionally persistent data structure. It's an append-only structure \(with some very minor caveats for 1. data owner _can_ overwrite to protect user sovereignty, and 2. the private segment layout is mutable at the block layer for raw performance\).

In order to share the latest version of your data with others, the root CID needs to be broadcast. WNFS works offline, but even in an online setting is fundamentally a distributed system. Knowing if your local version is ahead of the broadcast tree, or vice versa, if very important to guard against data loss.

{% hint style="info" %}
There is nothing special about the "broadcast" part of this setup. For all intents and purposes, this can \(and in future — _will\)_ be done with `n` machines \(since versions form a partial order / semilattice\).

As such, we will name the current pair being compared as "local" and "remote".
{% endhint %}

In a fully mutable setting, this can become tricky since data is dropped — you diverge _immediately_. You can work around this by comparing a history log \(as we do for the private section\), but a record of force-push is included in the tree to record that this is a forced , lossy point of synchronization. Persistent Merkle data structures have several nice properties that make approaching the problem more tractable.

## Comparison Algorithm

The basic comparison algorithm is the same in all cases, though some of the details change to maintain properties like security.

## Private

The private file system is straightforward: You are in sync if you have the same CID root for the data layer of the McTrie. Otherwise, do a direct multivalue merge. Attempt to do a fast-forward for your root node for your local pointer, and resolve any conflicts on that version \(see below\). Child resolution will happen as a natural course of use.

## Public

In all cases, we can think of the history as a set of CIDs \(only how they're stored is different\). If we add order \(a list instead of a set\), we can additionally tell where we diverged. There are four possible states:

1. In sync \(the heads are equal\)
2. Ahead of remote \(remote's head CID is contained in local's history\)
3. Behind remote \(local's head CID is contained in remote's history\)
4. Diverged with a shared history \(local and remote share a common ancestor\)

Or, if you prefer:

```haskell
data VersionOrder
  = InSync
  | AheadOfRemote
  | BehindRemote
  | DivergedAt CID
```

To give us a base case, we consider the genesis filesystem to be blank in all cases \(`QmbFMke1KXqnYyBBWxB74N4c5SBnJMVAiMNRcGu6x1AwQH`\). From intuition: every file system began blank before we added something to it.

Once we have the history, we can walk back one at a time, looking for the head CID of the other system. In principle we can do this one at a time \(`O(m + n)`\), but for performance we do this simultaneously on both local and remote file systems or use a Bloom filter cache for`O(2 * min(m, n))` run time. This can be further performance optimized, but this gets us a large part of the way there.

```haskell
-- newest to oldest
localHistory  = [localCID0,  localCID1,  localCID2]
remoteHistory = [remoteCID0, remoteCID1, remoteCID2, remoteCID3]

compareHistories :: [CID] -> [CID] -> VersionOrder
compareHistories locals remotes = innerCompare locals remotes [] mempty

innerCompare :: [CID] -> [CID] -> [CID] -> BloomFilter -> VersionOrder
innerCompare [] [] _ _ = InSync
innerCompare [] _  _ _ = DivergedAt genesis
innerCompare _  [] _ _ = DivergedAt genesis
innerCompare (local : moreLocal) (remote : moreRemote) checked cache =
  case (local == remoteHead, remote == localHead) of
    (True, True) -> 
      InSync
      
    (True, False) -> 
      AheadOfRemote
      
    (False, True) -> 
      BehindRemote
      
    (False, False) ->
      case filter alreadySeen [local, remote] of
        []        -> innerCompare moreLocal moreRemote checked' cache'
        (cid : _) -> DivergedAt cid
                
  where
    alreadySeen cid =
      cache `Bloom.conatins` cid && checked `List.contains` cid
  
    checked' = 
      (local : remote : checked)
              
    cache' = 
      cache
        |> BloomFilter.insert remote
        |> BloomFilter.insert checked
```

## Reconciliation

Merging in the case where one copy is strictly ahead or behind is straightforward: use the most recent version.

### Public Merge

WNFS has functional persistence, and this confluent history. Our Merklized layout forces  single merge point for all branches. Merges are associative, and we need a consistent order. We pick the latest CIDs for each branch, order them numerically lowest-to-highest. Working recursively bottom-up:

* Files: select a file \(default: pick the highest CID\)
* Directories: merge links by name
  * Defaults to resurrecting deleted links from one branch

When all branches are merged, publish a merge node that includes previous links to the heads of all branches under the key `mergeHistory`. Metadata is also map-merged.

The `previous` link for files that were directly select are pointed at again \(i.e. the one without the newly created `mergeHistory`\).

When walking back the history, the default behaviour is to take the `previous` link. Alternate paths may be taken if the agent prefers \(e.g. when doing error correction or searching for previous versions of files\). This can also be linearlized at runtime by any number of algorithms \(e.g. sequenced one branch after another, or interleaved by version number since divergence\).

### Private Merge

This is largely part of the regular operation of the private DAG, since we are always attempting to make progress by fast-forwarding. The merging user may not have access to previous versions of a file. Here a best-effort approach is taken: take the existing pointer, fast forward, and attempt to reconcile per the public merge. Due to secrecy constraints, the private file system does have a stronger bias towards LWW behaviour since there are cases where the reconciling agent may not have access to older history to do merges with lower-versioned nodes.

## WNFS Root

The root of the file system itself is designed to be very flexible, and support many different versioning methods below it, specifically:

* Structural \(`public`\)
* Log \(`private` for security\)
* Mutable \(`pretty` & `shared` since they're not versioned\)

As such, you need to look at  the sections themselves to determine priority. If one section is ahead of remote, and the other is behind remote, then this is considered to have diverged, and user intervention is required. This is actually not as bad as it sounds, since the actual data content would be the same even if comparing a versioned root. It feels off because we're treating the sections differently, but they're functionally equivalent.

## Where to Find History

### Structural Versioning

This is the most intuitive: walk the tree backwards along the `previous` links. This can be done lazily.

### Version Log

This style is to keep a log of versions identifiers, and walk down those lists. The logs don't need to be CIDs per se — any bijective mapping will do.

For instance, to avoid revealing correlated items on the private file system, we run the CIDs through a SHA256 first. This makes it impossible to discover the contents of each version unless you already have the CID. You don't learn any new information, since direct structural  analysis gives the same information if you have all previous versions available.

