# Data Layer

{% hint style="info" %}
This section describes how the `/private` segment would look to an **unauthorized** user. This is how the data is stored and propagated through the network only.
{% endhint %}

## Encryption

This layer is completely agnostic about file contents. By default, encyption is done via 256-bit AES-GCM \(via the [WebCrypto API](https://www.w3.org/TR/WebCryptoAPI/)\) but in principle can be done with any cipher \(e.g. [AES-SIV-GCM, ChaCha20-Poly1305](https://soatok.blog/2020/07/12/comparison-of-symmetric-encryption-methods/)\). Everything described below is compatible with any symmetric cipher.

{% hint style="info" %}
To see more about what is found _inside_ an SNode when unencrypted, please see the Private File Layer section.
{% endhint %}

## Secure Content Tree

Unlike the public file system DAG, the private file system is stored as a tree. More specifically, this is a SHA256-based [Modified Merkle Patricia Tree \(MMPT\)](https://eth.wiki/en/fundamentals/patricia-tree), with a branching factor of 16. The weight was chosen to balance search depth with Merkle witness size, caching, and concurrent merge performance. This tree can hold over a million elements in 5 layers.

As we will explore in later sections, collisions are not possible in this tree thanks to content addressing, so clients can aggressively cache intermediate nodes. Insertions have worst-case performance of O\(log n\). This is an append-only structure, so deletions are not supported. Practically, merges will overwhelmingly contain shared intermediate nodes and leaves, but the worst-case for a heavily diverged tree is linear relative to the smaller tree. Being a prefix tree, there are no rotations, so merging concurrently is straightforward.

$$
\begin{array} {|r|r|}\hline Lookup & O(log\ n) \\ \hline Insert & O(log\ n) \\ \hline Merge & O(min(n, m)) \\ \hline Delete & ⊥ \\ \hline  \end{array}
$$

The prefixes are not the CIDs of the data, but rather the namefilter \(see relevant section\). CIDs are kept as leaves in this tree, but all intermediate nodes refer to the set of \(hashed\) keys used for access control. Intermediate nodes are lightweight and SHOULD be aggressivley cached.

This structure emulates a hashtable of shape `Hash Namefilter -> Namefilter -> [CID]`. Terminal namefilter nodes may have multiple child CID leaves.

### Concurrency

This is a concurrent tree. Many contexts may be updating it at the same time without the ability to communicate directly \(e.g. network partition\). The namefilters themselves may have collisions, but the leaves cannot since they are hashes of the actual content. Conceptually, the same CID may live at multiple names in the MMPT, though this is extremely unlikely in practice.

Being an append-only data structure, merging in absence of namespace conflicts is very straightforward: place the new names in their appropriate positions in the tree. This can be done in high-parallel to further improve runtime performance.

#### Anti-Entropy & Fractional Indexing

In the case of namespace conflicts, store both leaves. In absence of other selection criteria \(such as hardcoded choice\), pick the lowest \(in binary\) CID.

IMAGE HERE

There is no need to manually track primacy. 

## Namefilters

Each SNode is stored not with a human readable name, but with a _namefilter_. This is a [generalized combinatoric accumulator](https://www.jstage.jst.go.jp/article/transinf/E91.D/5/E91.D_5_1489/_pdf/-char/en) \(GCA\), which in turn is essentially the Bloom construction of the better known [Nyberg hash accumulator](https://link.springer.com/content/pdf/10.1007%2F3-540-60865-6_45.pdf).

### Construction

Namefilters are _not_ a content address. They are based on the _keys_ used to construct that path. This is important for validating if a namefilter is allowed to be constructed \(via UCAN, see below\). They are essentially a set of hashes of the keys used to encrypt this node and all of the parents in the cyptree. The set-like property of forgetting the order is important: it should be very hard \(read: impossible except edge cases\) to infer the hierarchical relationship between any two nodes.

### Parameters

Namefilters are 2048-bit Bloom filters, encoded in [base64URL](https://datatracker.ietf.org/doc/html/rfc4648#section-5), yielding a consistent 32-character UTF-8 string. These hold 47 path segments, and achieve a one-in-a-billion false positive rate with 30 hashes. Formally:

$$
n = 47\\
p = 0.000000001\\
m = 2048\\
k = 30
$$

If required, doubling `n` and `m` leaves `p` and `k` constant. See [here for pretty graphs](https://hur.st/bloomfilter/?n=47&p=&m=2048&k=30) \(useful for parameter tuning, verified manually\).

#### Private Versioning

WNFS is a persistent, versioned file system. Including the version is essential for many parts of the system \(seen throughout the rest of this section\). In principle this can be any counter, including simple natural numbers, depending on the design goals of the broader system.

WNFS uses a forward secret positional hash clock, which is described in its own section.

This version is hashed and added to the private namefilter.

#### Bare / Unsaturated Namefilter

The bare namefilter for any node is the parent's bare namefilter plus the current node's read key. This bare namefilter is passed down to the child SNodes and encrypted along with other header information.

The root node has no parent, so its bare namefilter is merely the SHA-256 hash of its key placed in a Bloom filter. A child node is passed its parent's bare namefilter, and includes it with the SHA-256 of its key to generate its namefilter.

```haskell
bareParent = 0x01010101 -- parent, unless is root
currentKey = sha256(aesKey)
version    = sha256(hashClock)
bare       = bareParent .|. current .|. version
```

#### Hamming Saturation

By default, Bloom filters admit \(roughly\) how many elements they contain, and are relatively easy to correlate by their Hamming distance. To work around this issue, namefilters deterministically saturate the remaining space, filling roughly _half_ of the available filter, while maintaining a very low false positive rate. The idea is to fill the namefilter with a constant Hamming weight, but still be easily constructable by someone with the bare namefilter.

#### Pseudocode Algorithm

Saturation is then achieved by iteratively hashing the filter into its successor until a certain number of bits are set to 1.

```haskell
makeNameFilter :: BloomFilter -> BloomFilter
makeNameFilter bare = saturateTo 1410 aesKey bare

saturateTo :: Natural -> SHA256 -> BloomFilter -> BloomFilter
saturateTo threshold hashSeed namefilter =
  case (isSaturated namefilter, isSaturated namefilter') of
    (True, _) ->
      namefilter
      
    (_, True) ->
      -- Err on the side of slightly too few bits
      if inTollerance namefilter'
        then namefilter'
        else namefilter
        
    _ ->
      saturateTo threshold hashSeed' namefilter'
    
  where
    saturatedBy filter = Binary.sum filter - threshold
    isSaturated filter = saturatedBy filter >= 0
    
    -- 10 = k / 3, where k is number of bits per entry
    inTollerance filter = saturatedBy filter <= 10
    
    namefilter' = namefilter .|. toBloom hashSeed'
    hashSeed'   = sha256 hashSeed -- Recursively hash
```

In this way, we can deterministically generate very different looking filters for the same node, varying over the version number. The base filter stays inside the longer structure, . With an appropriately configured filter, this provides multiple features:

* Privacy
  * File names are never exposed
  * Statistical methods may be able to reveal probable DAG structures
* Deterministic pointers to the future
  * O\(log n\) search for updated nodes
* Minimal knowledge write access verification 
  * A UCAN + a hash of the read key to the highest node the user can write to
  * Match on cryptographically blind set membership

### UCAN

dsa

### Design Considerations

GCAs were chosen over other [arguably more sophisticated](https://www.fim.uni-passau.de/fileadmin/dokumente/fakultaeten/fim/forschung/mip-berichte/MIP_1210.pdf) options for three main reasons reasons: witness side, raw performance, and ease of implementation for web browsers. For example, we were unable to find a widely-used RSA accumulator library on NPM or Crates, but implementing a GCA is very straightforward.

Due to distinguishability, GCAs potentially leak some information about related, deeply-nested sibling nodes as the Hamming approaches zero. Our namefilter GCAs have a cardinality of 47 by default, which is much deeper than most file paths.

We considered using XOR or Cuckoo filters instead of class Bloom filters. XOR is very close to the theoretic efficiency limit, but is very new and the library untested. Cuckoo filters would provide around an additional 4 path segments with the same false-positive rate, but we lose the single-bit-collision of Bloom filters which is actually an advantage for obfuscation.

## SNode Content

An SNode that has been secured in this way is called an ”secure virtual node”. The contents of these nodes is largely the same as their plaintext counterparts, plus a key table for their children.

The core difference is the encrypted storage \(protocol layer\), and secrecy of the key used to start the decryption process. The key is always external to the ENode, and its not aware of whch key was used to create it. Here at the protocol layer, we are not directly concerned with the contents.

```haskell
data EncryptedNode 
  = EncryptedNode CID -- simple!

data ETree 
  = ETreeNode 
  | ETreeLeaf

data ETreeLeaf 
  = ETreeLeaf CID

data ETreeNode = ETreeNode
  { zero :: (HashFilter512, ETree) -- NOTEThese are *sides*, and may terminate directly
  , one  :: (HashFilter512, ETree)
  }
```

## SNode Naming

Files names are a consistent legth: 512-bits. They are XOR filters, and have a consistent number of bits flipped \(e.g. 320\). For instance \(expressed in base58\):

```text
yP4cqy7jmaRDzC2bmcGNZkuQb3VdftMk6YH7ynQ2Qw4zktKsyA9fk52xghNQNAdkpF9iFmFkKh2bNVG4kDWhsok
```

The method of generating these names is explained in the decrypted node section.

## Node Cache

{% hint style="info" %}
It’s concievable that this structure could be implemented ”natively” in IPFS by using the `Links` array, but would require investigating the bounds and performance characteristics of the protocol itself.

Another, option would be to store these as records in a SQLite database written to WNFS. This will probaby has better performance characteristics, even if it only has one table. It will also likely be able to do very efficient substring matching on keys \(file names\). Also:

> SQLite Is Public Domain

If SQLite works for our use case, we can ignore the hashing and whatnot below to better facilitate lookups by string match.
{% endhint %}

Currently traversing an MPT over the network of \(max\) depth 512 can take a while. Following paths in a decrypted node \(see below\) requires doing a large number of lookups, and having an in-memory structure to do this with speeds things up significantly.

A totally flat, append-only namespace with unique names has a number of nice properties. For one, it can be represented as a simple sorted array, which we do here.

{% hint style="warning" %}
Please note that this cache is just that: a cache. It can be destroyed and reconstituded at any time \(for example if it were to become corrupted\). That said, given that it’s publicly viewable, a validator can check that it conforms to the updates in the MPT.
{% endhint %}

The file namespace is larger than we could ever use. It includes many redundant bits to aid in private access control \(more below\), but need not exist in this lookup table. Instead we use a SHA-256 to reduce the size by nearly half in the hash table. Between the hash and CID, each record occupies ~64 bytes \(depending on generation of CID\). This table can store ~15k entries / MB. Most of the underlying blocks will stay the same size, so syncing updates is very efficient in the normal case.

```text
sha256(name)cid
```

There is additionally a compact cache, stored as a simple DSV file. Since the filenames are a consistent length, we don’t need a delimiter between the name and CID. As an example, with entries separated by newlines.

```text
2fFhPSYcgauRHumcQJqLvTxALipgmRLrAyYMDgmDVHU9QmfYStuhL72tdXoQWzicdzEehYaeXvhUNCrawBEWNP7DYX
33UrmA5tdPd4m97gB8FxRnwrErd3z2iKnU1W87zJqkMCQmetjBvK1M7STBSgauk1WaLHhzRG6mZpMeWZpEjYJXZcBi
7jRXo2LwMyUpgUuzkiKdNzFV4ZSZCbg5hjd3Ka7zsap5QmXvdZoqpPbsN6UQHomFtMiCm8C4VZZb8KBBUBapEB8LHP
9EHKSbWZQfgRtCowkNtQosmC6CeQajvpvUTK4zJixjEKQmW2WQi7j6c7UgJTarActp7tDNikE4B2qXtFCfLPdsgaTQ
```

An update to this is simply adding an entry at the correct \(ordered\) position in the file:

```bash
2fFhPSYcgauRHumcQJqLvTxALipgmRLrAyYMDgmDVHU9QmfYStuhL72tdXoQWzicdzEehYaeXvhUNCrawBEWNP7DYX
33UrmA5tdPd4m97gB8FxRnwrErd3z2iKnU1W87zJqkMCQmetjBvK1M7STBSgauk1WaLHhzRG6mZpMeWZpEjYJXZcBi
7jRXo2LwMyUpgUuzkiKdNzFV4ZSZCbg5hjd3Ka7zsap5QmXvdZoqpPbsN6UQHomFtMiCm8C4VZZb8KBBUBapEB8LHP
7B275X26t46NXcgxxBQc1x5QQHx9rULSn6aBb7yrc9b5QmfYStuhL72tdXoQWzicdzEehYaeXvhUNCrawBEWNP7DYX # New!
9EHKSbWZQfgRtCowkNtQosmC6CeQajvpvUTK4zJixjEKQmW2WQi7j6c7UgJTarActp7tDNikE4B2qXtFCfLPdsgaTQ
```

