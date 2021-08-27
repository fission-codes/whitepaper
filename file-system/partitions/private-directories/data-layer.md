# Data Layer

{% hint style="info" %}
This section describes how the `/private` partition would look to an **unauthorized** user. This is how the data is stored and propagated through the network only.
{% endhint %}

## Encryption

This layer is completely agnostic about file contents. By default, encryption is done via 256-bit AES-GCM \(e.g. via the [WebCrypto API](https://www.w3.org/TR/WebCryptoAPI/)\) but in principle can be done with any cipher \(e.g. [AES-SIV-GCM, ChaCha20-Poly1305](https://soatok.blog/2020/07/12/comparison-of-symmetric-encryption-methods/)\). Everything described below is compatible with any symmetric cipher.

{% hint style="info" %}
To see more about what is found _inside_ an SNode when unencrypted, please see the Private File Layer section.
{% endhint %}

## Secure Content Prefix Tree

Unlike the public file system DAG, the private file system is stored as a tree. More specifically, this is a SHA256-based HAMT with a branching factor of 1024. The weight was chosen almost entirely to limit the number of network calls required to sync the index from a new node. Network costs dominate in this use case, so it's worth paying the cost to local updates and cached nodes. Assuming that we can make sibling requests in parallel, lets us sync an index over a million elements in a two round trips, since the HAMT would be two levels deep

```javascript
// proto3

message SBitmap { // Feels a bit out of hand?
  fixed64 one = 1;
  fixed64 two = 2;
  fixed64 three = 3;
  fixed64 four = 4;
  fixed64 five = 5;
  fixed64 six = 6;
  fixed64 seven = 7;
  fixed64 eight = 8;
  fixed64 nine = 9;
  fixed64 ten = 10;
  fixed64 eleven = 11;
  fixed64 twelve = 12;
  fixed64 thirteen = 13;
  fixed64 fourteen = 14;
  fixed64 fifteen = 15;
  fixed64 sixteen = 16;
}

message SLink {
  enum SLinkType {
    SINDEX = 0;
    LONEVALUE = 1;
    MULTIVALUE = 2;
  }

  bytes cid = 1;
  SLinkType type = 2 [default = 0];
  uint64 size = 3;
}

message SIndex {
  required SBitmap bitmap = 0;
  repeated PBLink  links = 1;
}

// Example

{
  "data": 0b1000010000000010000, // ...but much longer. The HAMT bitmap.
  "links": [
    {
      "cid": "bafkreifjjcie6lypi6ny7amxnfftagclbuxndqonfipmb64f2km2devei4",
      "size": 100,
      "type": "SINDEX"
    },
    {
      "cid": "bafkreigtloekmii6hc7ngwg5msi3wwrem6zrgtfgvfwgerkdn3wgopsfny",
      "size": 99999,
      "type": "MULTIVALUE"
    },
    {
      "cid": "bafkreibzj64sbgo6it25xmlopk2ejzxkkpxug47oeoiy7iqsshfklzmzfq",
      "size": 42,
      "type": "SINDEX"
    }
  ]
}

// Lonevalue

{
  "Data": 
}

// Multivalue

{
  "Data": 0xbcb8d227bcf681aa4a8b580bfd07563b465c7e, // ...but much longer. The complete namefilter.
  "Links": [
    // File -- Alice's Variant
    {
      "hash": CID("bafkreifrsmmay6kv3iava6k6uwszux7wrh4b4oaaxedrd4eavav65avbuq"),
      "Tsize": 123456,
      // Name = sha256(author A)
      "Name": "node#ac943e8fb0b5e8d124660e0c3ab828e8f9faf09512a637fd76fc7073c305e6fa"
    },
    // UCAN -- Alice's Variant
    {
      "hash": CID("bafkreiez5l4lbmxlryxx3apxgqblbjmspq67lexqgwqpfnuthoz7zm7hkm"),
      "Tsize": 123456,
      // Name = same as associated file
      "Name": "ucan#ac943e8fb0b5e8d124660e0c3ab828e8f9faf09512a637fd76fc7073c305e6fa",
    },
    // File -- Bob's Variant
    {
      "hash": CID("bafkreicvjupn575izcorvjlkgowpksvd3iais2hagtxnozdsq2lul2ops4"),
      "Tsize": 13579,
      // Name = sha256(author A)
      "Name": "node#3f2f6b90b3b50ceb311f72be0a753a2501d516030a03689e28fa6c4a25cbd957"
    },
    // UCAN -- Bob's Variant
    {
      "hash": CID("bafkreiduk3iramyel3faerctu4dixrmclm2jxlcbgxq42mazaenux5ariq"),
      "Tsize": 24680,
      // Name = same as associated file
      "Name": "ucan#3f2f6b90b3b50ceb311f72be0a753a2501d516030a03689e28fa6c4a25cbd957"
    }
  ]
}
```

You may note a complete lack of filenames in th

The hases are not the CIDs of the data, but rather the namefilter \(see relevant section\). CIDs are kept as leaves in this tree, but all intermediate nodes refer to the set of \(hashed\) keys used for access control. Intermediate nodes are lightweight and SHOULD be aggressively cached.

This structure emulates a hash table of shape `Hash Namefilter -> (Namefilter, [CID])`. Terminal namefilter nodes may have multiple child CID leaves.

## Concurrency

This is a concurrent tree. Many contexts may be updating it at the same time without the ability to communicate directly \(e.g. network partition\). The namefilters themselves may have collisions, but the leaves cannot since they are hashes of the actual content. Conceptually, the same CID may live at multiple names in the McTrie, though this is extremely unlikely in practice.

Being an append-only data structure, merging in absence of namespace conflicts is very straightforward: place the new names in their appropriate positions in the tree. This can be done in high-parallel to further improve runtime performance.

> Last, let's consider the case where it is truly ambiguous what order to apply changes using the same example but with different visibility \[...\] Here, we have no "right" answer. Alice and Bob both have made changes without the other's knowledge and now as they synchronize data we have to decide what to do. Importantly, there isn't a _right_ answer here. Different systems resolve this in different ways.
>
> ~[ Hypermerge's Architecture Documentation](https://github.com/automerge/hypermerge/blob/master/ARCHITECTURE.md)

#### Multivalues-else-LWW

In the case of namespace conflicts, store both leaves. In absence of other selection criteria \(such as hard-coded choice\), pick the highest \(in binary\) CID. In other words, pick the longest causal chain, and deterministically select an arbitrary element when the event numbers overlap. This is a variant of multivalues, with nondestructive last-writer-wins \(LWW\) semantics in the case of a longest chain.

![Multivalue example, https://bartoszsypytkowski.com/operation-based-crdts-registers-and-sets/](../../../.gitbook/assets/multi-value-register-timeline.png)

By default, WNFS automatically picks the the highest revision, or in the case of multiple values at a single version, the highest namefilter number. Here is one such example, where the algorithm would automatically chose `QmbX21...` as the default variant. The user can override this choice by pointing at `Qmr18U...` from the parent directory, or directly in the link. This is related to [preferred edges in a link/cut tree](https://en.wikipedia.org/wiki/Link/cut_tree#Preferred_paths).

![](../../../.gitbook/assets/screen-shot-2021-06-02-at-20.04.00.png)



Any observer may perform a merge to produce a valid WNFS structure. If an agent with read/write access performs such a merge, it is strongly recommended to do the following:

1. Lazily root new nodes \(see "Lazy Progress" below\)
2. Merge conflict branches, either merge the contents semantically, or simply select a preferred winner, and writing that as a new revision

## Secret VNode \(SNode\) Content

An SNode that has been secured in this way is called a ”secure virtual node”. The contents of these nodes is largely the same as their plaintext counterparts, plus a key table for their children.

The core difference is the encrypted storage \(protocol layer\), and secrecy of the key used to start the decryption process. The key is always external to the SNode, and its not aware of which key was used to create it. Here at the protocol layer, we are not directly concerned with the contents.

```haskell
data McBranch
  = McLeaf CID Namefilter FileContent
  | McTree CID

data McTree = McTree
  { x0 :: Maybe SNode -- NOTE May terminate "early" if no 
  , x1 :: Maybe SNode
  , -- ...
  , xE :: Maybe SNode
  , xF :: Maybe SNode
  }
```

## Cooperative Rooting

Since the ratchets and namefilters are deterministic, we can look up a version in constant time from the McTrie. Below we go into greater detail about how progress is ensured, but is relevant to lookup.

If you have a pointer to a particular file, there is no way of knowing that you have been linked to the latest version of a node. The information that you do have includes everything that you need to construct a name filter.

* The current node’s identity nonce
* The ratchet of the current node \(stored in the node\)
* This parent's bare name filter \(stored in the node\)

The user must always ”look ahead” to see if there have been updates to the file since they last looked. The three most common scenarios are that:

1. No changes have been made
2. There have been been a small number of changes
3. There have substantial changes since

### Fast Forward Mechanism

#### Simplified

To balance these scenarios, we progressively check for files at revision `r + 2^n` , where `r` is the current revision, and `n` is the search index. First we check the next revision. If it does not exist, we know that we have the latest version. If it does exist, check `r + 2`, then `r+4`, `r+8` and so on. Once there’s a missing version, perform a binary search. For example, if looking at a node at revision 42 that has been updated 123 times since your last recorded pointer, it takes 14 checks \(roughly `O(2 log n)`\) to find the latest revision.

![](../../../.gitbook/assets/screen-shot-2021-06-03-at-23.46.07%20%281%29.png)

#### Search Attack Resistance

A fully deterministic lookup mechanism is open to an attack where the malicious user only writes nodes that are known to be on the lookup path, forcing a linear lookup time against a large number of nodes. To work around this, we add noise to the lookup values while looking performing large jumps: 

```haskell
-- Pseudocode
fastForward :: forall m . MonadRandom m => SpiralRatchet -> McTrie -> m Node
fastFoward rachet store = findUpperBound 0 0
  where
    -- Start by looking for the first missing element to establish a max bound
    findUpperBound :: Natural -> Natural -> m Node
    findUpperBound latestIndex exponent = do
      index <- noisyIndex latestIndex (latestIndex - 2^exponent)
      case findIn store (rachet `advanceBy` index) of
        Just _  -> findFirstMiss (exponent + 1)
        Nothing -> narrow 2^exponent index
          
    -- Noisy split search
    narrow :: Natural -> Natural -> m Node
    narrow floorIndex ceilingIndex
      | floorIndex == ceilingIndex = return $ found floorIndex
      | otherwise                  = search floorIndex ceilingIndex
        
    search :: Natural -> Natural -> m Node
    search low high = do
      index <- noisyIndex floorIndex (round ((ceilingIndex - floorIndex) / 2))
      case findIn store (rachet `advanceBy` index) of
        Just _  -> narrow index high
        Nothing -> narrow low   index
        
    found :: Natural -> Node
    found index =
      case findIn store (rachet `advanceBy` index) of
        Just node -> node
        Nothing   -> error "Should not be possible"

    noisyIndex :: Natural -> Natural -> m Natural
    noisyIndex latestIndex diff =
      noise <- if diff < 1024 then pure 0 else rand 0 diff
      return $ latestIndex + diff - noise
```

## Lazy Progress

Anyone that can update a pointer can make permanent revision progress for themselves \(in `localStorage` or as a symlink in their FS\), or others if they have write access to this file system.

As the user traverses the private section \(down the Y-axis, across the X-axis\), they attempt to make progress in time \(forwards in the Z-axis\). If they find a node that’s ahead of a link, it updates that one link in memory. At the end of their search in 3-dimensions \(with potentially multiple updates\), they write the new paths to WNFS.

Note that they only need to do this with the paths that they actually follow! Progress in revision history does not need to be in lock step, and will converge over time.

Not all users with write access have the ability to write to the entire DAG. Writing to a subgraph is actually completely fine. Each traversal down a path will reach the most recently written node. The search space for that node is always smaller than its previous revisions, and can be further updated with other links or newer child nodes.

This contributes back collaboratively to the overall performance of the system for all users. If a malicious user writes a bad node, they can be overwritten with a newer revision by a user with equal or higher privileges. Nothing is ever lost in WNFS, so reconstructing all links in a file system from scratch is _possible_ \(though compute intensive\).

![Partial Rooting](../../../.gitbook/assets/screen-shot-2021-06-09-at-21.45.41.png)

![Complete Rooting](../../../.gitbook/assets/screen-shot-2021-06-09-at-21.47.39.png)

## Name Graveyard

The graveyard is a RECOMMENDED safeguard against writing to a file that will no longer be followed. It is a prefix tree containing hashes of all file descriptors that have been retired. McTrie merges must include all files even if they fail this check; it is only here as a convenience to prevent agents with imperfect knowledge to be aware that a particular path is no longer followed.

