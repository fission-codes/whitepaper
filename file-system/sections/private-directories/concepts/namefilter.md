# Namefilter / i-number

At the data layer, each secret node \(SNode\) is placed in a table and named with a namefilter \(AKA a deterministic i-number\). This is a [generalized combinatoric accumulator](https://www.jstage.jst.go.jp/article/transinf/E91.D/5/E91.D_5_1489/_pdf/-char/en) \(GCA\), which in turn is essentially the Bloom construction of the [Nyberg hash accumulator](https://link.springer.com/content/pdf/10.1007%2F3-540-60865-6_45.pdf).

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

#### File Descriptor

The identity of a file is a random 256-bit value. This fills the role of a standard file descriptor. This value must be present in the namefilter for write access to work \(see private file UCAN write semantics\).

#### Bare / Unsaturated Namefilter

The bare namefilter for any node is the parent's bare namefilter plus the current node's read key. This bare namefilter is passed down to the child SNodes and encrypted along with other header information.

The root node has no parent, so its bare namefilter is merely the SHA-256 hash of its identity hash placed in a Bloom filter. A child node is passed its parent's bare namefilter, and includes it with the SHA-256 of its key to generate its namefilter.

```haskell
bareParent = 0xabcdef -- parent, unless is root
currentKey = sha256(aesKey)
version    = sha256(hashClock)
bare       = bareParent .|. current .|. version
```

#### Private Versioning

WNFS is a persistent, versioned file system. Including the version is essential for many parts of the system \(seen throughout the rest of this section\). In principle this can be any counter, including simple natural numbers, depending on the design goals of the broader system.

WNFS uses a forward-secret spiral ratchet for versioning, which is described in its own section. This ratchet is hashed and added to the bare namefilter.

#### Hamming Saturation

Bloom filters admit \(roughly\) how many elements they contain, and are relatively easy to correlate by their Hamming distance. To work around this issue with obsfucation, namefilters deterministically saturate the remaining space, filling just over _half_ of the available filter, while maintaining a very low false positive rate. The idea is to fill the namefilter with a constant Hamming weight, but still be easily constructable by someone with the bare namefilter.

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

### Design Considerations

GCAs were chosen over other [arguably more sophisticated](https://www.fim.uni-passau.de/fileadmin/dokumente/fakultaeten/fim/forschung/mip-berichte/MIP_1210.pdf) options for three main reasons reasons: witness side, raw performance, and ease of implementation for web browsers. For example, we were unable to find a widely-used RSA accumulator library on NPM or Crates, but implementing a GCA is very straightforward.

Due to distinguishability, GCAs potentially leak some information about related, deeply-nested sibling nodes as the Hamming approaches zero. Our namefilter GCAs have a cardinality of 47 by default, which is much deeper than most file paths.

We considered using XOR or Cuckoo filters instead of class Bloom filters. XOR is very close to the theoretic efficiency limit, but is very new and the library untested. Cuckoo filters would provide around an additional 4 path segments with the same false-positive rate, but we lose the single-bit-collision of Bloom filters which is actually an advantage for obfuscation.

