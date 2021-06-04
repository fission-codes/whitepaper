# Data Layer

{% hint style="info" %}
This section describes how the `/private` segment would look to an **unauthorized** user. This is how the data is stored and propagated through the network only.
{% endhint %}

## Encryption

This layer is completely agnostic about file contents. By default, encyption is done via 256-bit AES-GCM \(via the [WebCrypto API](https://www.w3.org/TR/WebCryptoAPI/)\) but in principle can be done with any cipher \(e.g. [AES-SIV-GCM, ChaCha20-Poly1305](https://soatok.blog/2020/07/12/comparison-of-symmetric-encryption-methods/)\). Everything described below is compatible with any symmetric cipher.

{% hint style="info" %}
To see more about what is found _inside_ an SNode when unencrypted, please see the Private File Layer section.
{% endhint %}

## Secure Content Prefix Tree

Unlike the public file system DAG, the private file system is stored as a tree. More specifically, this is a SHA256-based [Modified Merkle Patricia Tree \(MMPT\)](https://eth.wiki/en/fundamentals/patricia-tree), with a branching factor of 16. The weight was chosen to balance search depth with Merkle witness size, caching, and concurrent merge performance. This tree can hold over a million elements in 5 layers.

As we will explore in later sections, collisions are not possible in this tree thanks to content addressing, so clients can aggressively cache intermediate nodes. This is an append-only structure, so deletions are not supported. 

Practically, merges overwhelmingly contain shared intermediate nodes and leaves, but the worst-case for a heavily diverged tree is linear relative to the smaller tree. Being a prefix tree, there are no rotations. Tree merging forms a monoid, so concurrent merges are straightforward.

$$
\begin{array} {|r|r|}\hline Lookup & O(log\ n) \\ \hline Insert & O(log\ n) \\ \hline Merge & O(min(n, m)) \\ \hline Delete & ⊥ \\ \hline  \end{array}
$$

The prefixes are not the CIDs of the data, but rather the namefilter \(see relevant section\). CIDs are kept as leaves in this tree, but all intermediate nodes refer to the set of \(hashed\) keys used for access control. Intermediate nodes are lightweight and SHOULD be aggressivley cached.

This structure emulates a hashtable of shape `Hash Namefilter -> Namefilter -> [CID]`. Terminal namefilter nodes may have multiple child CID leaves.

### Concurrency

This is a concurrent tree. Many contexts may be updating it at the same time without the ability to communicate directly \(e.g. network partition\). The namefilters themselves may have collisions, but the leaves cannot since they are hashes of the actual content. Conceptually, the same CID may live at multiple names in the MMPT, though this is extremely unlikely in practice.

Being an append-only data structure, merging in absence of namespace conflicts is very straightforward: place the new names in their appropriate positions in the tree. This can be done in high-parallel to further improve runtime performance.

> Last, let's consider the case where it is truly ambiguous what order to apply changes using the same example but with different visibility \[...\] Here, we have no "right" answer. Alice and Bob both have made changes without the other's knowledge and now as they synchronize data we have to decide what to do. Importantly, there isn't a _right_ answer here. Different systems resolve this in different ways.
>
> ~[ Hypermerge's Architecture Documentation](https://github.com/automerge/hypermerge/blob/master/ARCHITECTURE.md)

#### LWW & Multivalues

In the case of namespace conflicts, store both leaves. In absence of other selection criteria \(such as hardcoded choice\), pick the lowest \(in binary\) CID. In other words, pick the longest causal chain, and deterministically select an arbitrary element when the event numbers overlap. This is a mix of causal order last-writer-wins \(LWW\), and multivalues.

![Multivalue example, https://bartoszsypytkowski.com/operation-based-crdts-registers-and-sets/](../../../.gitbook/assets/multi-value-register-timeline.png)

By default, WNFS will automatically pick the the highest revision, or in the case of multiple values at a single version, the highest namefilter number. Here is one such exmaple, where the algorithm would automatically chose `QmbX21...` as the default variant. The user can override this choice by pointing at `Qmr18U...` from the parent directory, or directly in the link.

![](../../../.gitbook/assets/screen-shot-2021-06-02-at-20.04.00.png)

## Forward-Secret Spiral Ratchet

Every node in the MMPT is encrypted with a different key. This is done randomly for child nodes \(y-axis\), and deterministically with a cryptographic ratchet for increasing versions.

The basic idea for cryptographic ratchets is that repeatedly hashing a value creates a kind of forward-secret clock. When you start watching the clock, you can generate the hash for any arbitrary future steps, but not steps from prior to observation since that requires computing the SHA preimage.

SHA-256 is native to the WebCypto API, is a very fast operation, and commonly hardware accelerated. Anecdotally, running 10k recursive SHA-256s in Firefox on an Apple M1 completes in around 300ms. The problem with a single hash counter is threefold:

1. The root of the unencrypted tree updates with with every atomic operation, and thus accrues a lot of changes  
2. An actor may be many months since their last sync, and need to fast forward their clock by some huge number of elements  
3. Seeking ahead by 100ks or millions takes very noticeable time

We still want small step changes to very fast, but also be able to deterministically tune how quickly we jump ahead, without revealing previous counter hashes, while maintaining the same security properties as a single ratchet counter.

Positional counting does exactly this in the most common numeral systems. Our design involves three positions, two of which are bounded. We call these the large \(or epoch\), medium, and small numbers. 

Positionally this look like:

```text
large   medium  small
l*2^16  m*2^8   s*2^0

l = 0x600b56e66b7d12e08fd58544d7c811db0063d7aa467a1f6be39990fed0ca5b33
m = 0x5d58264a09dce1f2676e729d0ea1db4bf90b9be463d7fc1aa9b43b358e514599
s = 0xc8633540cabdf591e07918a2595964cc1b692d0f9392f079f2f110c08b67c6f4
```

The large and small are bounded at 256 elements. We achieve this by keeping a record of the max bound of these numbers. This is found by hashing that number 255 times \(0 is the unchanged value\). As we increment each number by taking its SHA256, we check if it matches the max bound. If it does, we increment the next-highest value, and reset the smaller values.

The metaphor is a spiral. You can ratchet one at a time, or deterministically skip to the start of the next ring in the spiral.

```haskell
data SpiralRatchet = SpiralRatchet
  { large     :: SHA256

  , medium    :: SHA256
  , mediumMax :: SHA256
  
  , small     :: SHA256
  , smallMax  :: SHA256
  }
  
exampleSR = SpiralRatchet
  { large     = 0x600b56e66b7d12e08fd58544d7c811db0063d7aa467a1f6be39990fed0ca5b33
  
  , medium    = 0x5d58264a09dce1f2676e729d0ea1db4bf90b9be463d7fc1aa9b43b358e514599
  , mediumMax = 0x5aae7b2b881d21863292a1556eafd2a3b21527f64f33c6fcc2beaa9d9cf1fe5f
  
  , small     = 0xc8633540cabdf591e07918a2595964cc1b692d0f9392f079f2f110c08b67c6f4
  , smallMax  = 0xb95c5d8851daff6204eb227f56d8c6af1c11a80d46d17eb0aa219a9d2ec109af
  }
```

Incrementing a larger value resets all smaller values. This is done by taking the SHA-256 of the \(one's\) complement. This cascades from larger to all smaller positions.

{% hint style="danger" %}
Please note that JavaScript's basic inverse function behaves unexpectedly. The `~` operator casts values to their one's compliment _integer_. This means that `~0b11 !== 0b00`, but rather `~0b11 === -4`. `-4` can not be directly expressed in JS binary, since it interprets binary notation as being a natural number only \(rather than its two's compliment\). To keep this from happening, use a toggle mask of equal length: `val ^ b1111...`.
{% endhint %}

The small and medium values are base-256. This means that the first position increments in ones \(2^0\), the second position increments in 256s \(2^8\), and the third position by 65,536s \(2^16\). Looking at a triple does not admit which number it is, only which number relative to the max bounds of the medium and small numbers. It is not possible to determine which of epoch \(large number\) you are in. Here is an example of a spiral ratchet being constructed from a seed value.

```haskell
seed       = 0x600b56e66b7d12e08fd58544d7c811db0063d7aa467a1f6be39990fed0ca5b33
large      = sha256 seed -- e.g. 0x8e2023cc8b9b279c5f6eb03938abf935dde93be9bfdc006a0f570535fda82ef8
  
mediumSeed = sha256 $ Binary.complement seed
medium     = sha256 mediumSeed
mediumMax  = iterate sha256 medium !! 255
   
smallSeed  = sha256 $ Binary.complement mediumSeed
small      = sha256 smallSeed
smallMax   = iterate sha256 small !! 256
```

Here is how to increment the ratchet by one:

```haskell
-- large (0..), medium (0-255), small (0-255)

toVersionHash :: SHA256
toVersionHash SpiralRatchet {..} = sha256 (large `xor` medium `xor` small)

advance :: SpiralRatchet -> SpiralRatchet
advance SpiralRatchet {..} =
  case (nextSmall == smallCeiling, nextMedium == mediumCeiling) of
    (False, _)    -> advanceSmall
    (True, False) -> advanceMedium
    (_,    True)  -> advanceLarge
    (False, True) -> error "PROBLEM (fix in smart constructor?)"

  where
    nextSmall  = sha256 small
    nextMedium = sha256 medium

    advanceSmall :: SpiralRatchet
    advanceSmall = SpiralRatchet {small = nextSmall, ..}

    advanceMedium :: SpiralRatchet
    advanceMedium =
      let
        resetSmall  = sha256 $ compliment nextMedium
        resetMedium = sha256 $ compliment nextLarge

      in
        SpiralRatchet
          { large      = large

          , medium     = nextMedium
          , mediumCeil = recursiveSHA 256 nextMedium

          , small      = resetSmall
          , smallCeil  = recursiveSHA 256 resetSmall
          }

    advanceLarge :: SpiralRatchet
    advanceLarge =
      let
        nextLarge   = sha256 large
        resetMedium = sha256 $ compliment nextLarge
        resetSmall  = sha256 $ compliment resetMedium

      in
        SpiralRatchet
          { large      = nextLarge

          , medium     = resetMedium
          , mediumCeil = recursiveSHA 256 resetMedium

          , small      = resetSmall
          , smallCeil  = recursiveSHA 256 resetSmall
          }
```

Many files will have few revisions. To prevent leaking this information, we take advantage of the fact that the setup starts at a random point in the ratchet:

```haskell
setup :: SpiralRatchet
setup = do
  largePre   <- random -- ratchet seed
  mediumSkip <- randomBetween 0 254
  smallSkip  <- randomBetween 0 254

  let
    large = sha256 largePre

    mediumPre = sha256 $ compliment largePre
    medium    = recursiveSHA mediumSkip mediumPre
    mediumMax = recursiveSHA 255 mediumPre

    smallPre = sha256 $ compliment mediumPre
    small    = recursiveSHA smallSkip smallPre
    smallMax = recursiveSHA 255 smallPre

  return SpiralRatchet {..}
```

## Namefilters

Each SNode is stored not with a human readable name, but with a _namefilter_. This is a [generalized combinatoric accumulator](https://www.jstage.jst.go.jp/article/transinf/E91.D/5/E91.D_5_1489/_pdf/-char/en) \(GCA\), which in turn is essentially the Bloom construction of the [Nyberg hash accumulator](https://link.springer.com/content/pdf/10.1007%2F3-540-60865-6_45.pdf).

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
bareParent = 0x01010101 -- parent, unless is root
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

## SNode Content

An SNode that has been secured in this way is called an ”secure virtual node”. The contents of these nodes is largely the same as their plaintext counterparts, plus a key table for their children.

The core difference is the encrypted storage \(protocol layer\), and secrecy of the key used to start the decryption process. The key is always external to the SNode, and its not aware of which key was used to create it. Here at the protocol layer, we are not directly concerned with the contents.

```haskell
data SecretNode 
  = SecretNode CID -- simple!

data STree 
  = STreeNode 
  | STreeLeaf

data STreeLeaf 
  = STreeLeaf CID

data STreeNode = STreeNode
  { zero :: (Namefilter, STree) -- NOTEThese are *sides*, and may terminate directly
  , one  :: (namefilter, STree)
  }
```

