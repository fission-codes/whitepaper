# Namefilter

At the data layer, each secret node \(SNode\) is placed in a table and named with a namefilter. This is a [generalized combinatoric accumulator](https://www.jstage.jst.go.jp/article/transinf/E91.D/5/E91.D_5_1489/_pdf/-char/en) \(GCA\), which in turn is essentially the Bloom construction of the [Nyberg hash accumulator](https://link.springer.com/content/pdf/10.1007%2F3-540-60865-6_45.pdf).

## Construction

Namefilters are _not_ a content address. They are based on the _keys_ used to construct that path. This is important for validating if a namefilter is allowed to be constructed \(via UCAN, see below\). They are essentially a set of hashes of the keys used to encrypt this node and all of the parents in the cyptree. The set-like property of forgetting the order is important: it should be very hard \(read: impossible except edge cases\) to infer the hierarchical relationship between any two nodes.

## Parameters

Namefilters are 2048-bit Bloom filters. These hold 47 path segments, and achieve a one-in-a-billion false positive rate with 30 hashes. Formally:

![](../../../../.gitbook/assets/screen-shot-2021-08-26-at-20.19.48.png)

If required, doubling `n` and `m` leaves `ε` and `k` constant. See [here for pretty graphs](https://hur.st/bloomfilter/?n=47&p=&m=2048&k=30) \(useful for parameter tuning, verified manually\).

### Hash Function

Many Bloom filter implementations are optimized for speed, not consistency. We have chosen the [XXH3](https://cyan4973.github.io/xxHash/) \(i.e. 64-bit\) algorithm. 

### Max Popcount / Hamming Saturation

Bloom filters admit \(roughly\) how many elements they contain, and are relatively easy to correlate by their Hamming distance. To work around this issue with obfuscation, namefilters deterministically saturate the remaining space, filling just under _half_ of the available filter, while maintaining a very low false positive rate. The idea is to fill the namefilter with a constant Hamming weight, but still be easily constructible by someone with the bare namefilter.

To satisfy these constraints, we have chosen a target saturation of 1019, with some tollerances. 1019 is chosen as it represents the 47 elements, yielding the lower bound false positive rate.

This is granted some tolerances: since every element takes _up to_ 30 bins, we don't know how many bits will overlap. As such, we need to find the overshoot of 1019 elements, and take the previous value. This requires limited backtracking.

## Bare / Unsaturated Namefilter

The bare namefilter for any node is the parent's bare namefilter plus the current node's read key. This bare namefilter is passed down to the child SNodes and encrypted along with other header information.

The root node has no parent, so its bare namefilter is merely the SHA-256 hash of its identity hash placed in a Bloom filter. A child node is passed its parent's bare namefilter, and includes it with the SHA-256 of its key to generate its namefilter.

```javascript
const bareParent = 0x5891b5b522d5df086d0ff0b110fbd9d21bb4fc7163af34d08286a2e846f6be03
const iNumber = 0xe258d248fda94c63753607f7c4494ee0fcbe92f1a76bfdac795c9d84101eb317
const versionedKey = sha256(spiralRatchet.toKey())
const bare = bareParent ^ inumber ^ versionedKey
```

## Private Versioning

WNFS is a persistent, versioned file system. Including the version is essential for many parts of the system \(seen throughout the rest of this section\). In principle this can be any counter, including simple natural numbers, depending on the design goals of the broader system.

WNFS uses a backward-secret spiral ratchet for versioning, which is described in its own section. This ratchet is hashed and added to the bare namefilter.

## Handling Collisions

There is an unlikely case where adding an element causes no change to the filter, and thus causes an infinite loop. Getting around this is straightforward: take the binary complement of the filter, hash that, and continue:

```javascript
if (filterBefore === filterAfter) {
  filterAfter.add(sha(complement(filterAfter.toBytes()))
} else {
  filterAfter
}
```

The naive approach is to check on every hash. Given that this is an _extremely_ unlikely situation, we can perform the check only in the slower portion of the algorithm, while checking the hamming weight / popcount. See the pseudocode section for an example.

### Algorithm \(in Pseudocode\)

```typescript
const max: number = 1019
const hashCount: number = 30

const saturate = (barefilter: NameFilter): NameFilter {
  // Get the lower bound of remaining elements
  const lowerBound = Math.floor((max - barefilter.popcount()) / hashCount)

  // Quickly jump to the lower bound
  let filter = barefilter
  for (i = 0; i < lowerBound; i++) {
    filter = filter.add(sha(filter.toBytes()))
  }

  // Step more slowly though until comparison reached
  return saturatedUnderMax(filter)
}

// Closest without going over
const saturateUnderMax = (filter: NameFilter): NameFilter {
  let newFilter = filter.add(sha(filter.toBytes()))
  if (filter === candidate) {
    // Change value deterministically to break out of infinite loop
    const breakout = complement(filter.toBytes())
    // Mutually recursive! Start again with new popcount
    saturate(filter.add(sha(breakout)))
  }
  
  return popcount(newFilter) > max ? filter : saturatedUnderMax(newFilter)
}
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

## Design Considerations

GCAs were chosen over other [arguably more sophisticated](https://www.fim.uni-passau.de/fileadmin/dokumente/fakultaeten/fim/forschung/mip-berichte/MIP_1210.pdf) options for three main reasons: witness side, raw performance, and ease of implementation for web browsers. For example, we were unable to find a widely-used RSA accumulator library on NPM or Crates, but implementing a GCA is very straightforward.

Due to distinguishability, GCAs potentially leak some information about related, deeply-nested sibling nodes as the Hamming approaches zero. Our namefilter GCAs have a cardinality of 47 by default, which is much deeper than most file paths.

We considered using XOR or Cuckoo filters instead of class Bloom filters. XOR is very close to the theoretic efficiency limit, but is very new and the library untested. Cuckoo filters would provide around an additional 4 path segments with the same false-positive rate, but we lose the single-bit-collision of Bloom filters which is actually an advantage for obfuscation.

## Calculations

Because it's important to show your work

### False Positive Probability \(FPP\)

![](../../../../.gitbook/assets/screen-shot-2021-08-26-at-20.19.41.png)

Which is \(very\) roughly 8/10B, or a little under 1/1 billion

### Optimal Number of Hash Functions \(k\)

![](../../../../.gitbook/assets/screen-shot-2021-08-26-at-20.19.38.png)

### Optimal Number of Elements \(n\) for Hash Count

![](../../../../.gitbook/assets/screen-shot-2021-08-26-at-20.19.35.png)

### Optimal Popcount \(X\)

This is the formula to estimate the number of elements in a given Bloom filter \([source](https://en.wikipedia.org/wiki/Bloom_filter#Approximating_the_number_of_items_in_a_Bloom_filter)\)

![](../../../../.gitbook/assets/screen-shot-2021-08-26-at-20.21.56.png)

