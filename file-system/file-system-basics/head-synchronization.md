# Head Synchronization

The Web Native File System is a functionally persistent data structure. It's an append-only structure \(with some very minor caveats for 1. data owner _can_ overwrite to protect user sovereignty, and 2. the private segment layout is mutable at the block layer for raw performance\).

In order to share the latest version of your data with others, the root CID needs to be broadcast. WNFS works offline, but even in an online setting is fundamentally a distributed system. Knowing if your local version is ahead of the broadcast tree, or vice versa, if very important to guard against data loss.

{% hint style="info" %}
There is nothing special about the "broadcast" part of this setup. For all intents and purposes, this can \(and in future — _will\)_ be done with `n` machines \(since versions form a partial order / semilattice\).

As such, we will name the current pair being compared as "local" and "remote".
{% endhint %}

In a fully mutable setting, this can become tricky since data is dropped — you diverge _immediately_. You can work around this by comparing a history log \(as we do for the private section\), but a record of force-push is included in the tree to record that this is a forced , lossy point of synchronization. Persistent Merkle data structures have several nice properties that make approaching the problem more tractable.

## Comparison Algorithm

The basic comparison algorithm is the same in all cases, though some of the details change to maintain properties like security.

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

To give us a base case, we consider the genesis filesystem to be blank in all cases \(`Qmc5m94Gu7z62RC8waSKkZUrCCBJPyHbkpmGzEePxy2oXJ`\). From intuition: every file system began blank before we added something to it.

Once we have the history, we can walk back one at a time, looking for the head CID of the other system. In principle we can do this one at a time \(`O(m + n)`\), but for performance we do this simultaneously on both local and remote file systems \(`O(2 * min(m, n))`\)

```haskell
-- newest to oldest
localHistory  = [localCID0,  localCID1,  localCID2]
remoteHistory = [remoteCID0, remoteCID1, remoteCID2, remoteCID3]

compareHistories :: [CID] -> VersionOrder
compareHistories [] [] = InSync
compareHistories [] _  = BehindRemote
compareHistories _  [] = AhedOfRemote
compareHistories locals@(localHead : _) remotes@(remoteHead : _) =
  innerCompae locals remotes
  where
    innerCompare [] [] = InSync
    innerCompare [] _  = DivergedAt genesis
    innerCompare _  [] = DivergedAt genesis
    innerCompare (localFocus : moreLocal) (remoteFocus : moreRemote) =
      case (localFocus == remoteHead, remoteFocus == localHead) of
        (True, True)   -> InSync
        (True, False)  -> AheadOfRemote
        (False, True)  -> BehindRemote
        (False, False) -> innerCompare moreLocal moreRemote
```

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

