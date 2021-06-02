# Data Layer

{% hint style="info" %}
This section describes how the `/private` segment would look to an **unauthorized** user. This is how the data is stored and propagated through the network only.
{% endhint %}

## Encryption

This layer is completely agnostic about file contents. By default, encyption is done via 256-bit AES-GCM \(via the [WebCrypto API](https://www.w3.org/TR/WebCryptoAPI/)\) but in principle can be done with any cipher \(e.g. [AES-SIV-GCM, ChaCha20-Poly1305](https://soatok.blog/2020/07/12/comparison-of-symmetric-encryption-methods/)\). Everything described below is compatible with any symmetric cipher.

{% hint style="info" %}
To see more about what is found _inside_ an SNode when unencrypted, please see the Private File Layer section.
{% endhint %}

## Namefilters

Each SNode is stored not with a human readable name, but with a _namefilter_. This is a [generalized combinatoric accumulator](https://www.jstage.jst.go.jp/article/transinf/E91.D/5/E91.D_5_1489/_pdf/-char/en) \(GCA\), which in turn is essentially the Bloom construction of the better known [Nyberg hash accumulator](https://link.springer.com/content/pdf/10.1007%2F3-540-60865-6_45.pdf).

### Design Considerations

GCAs were chosen over other [arguably more sophisticated](https://www.fim.uni-passau.de/fileadmin/dokumente/fakultaeten/fim/forschung/mip-berichte/MIP_1210.pdf) options for three main reasons reasons: witness side, raw performance, and ease of implementation for web browsers. For example, we were unable to find a widely-used RSA accumulator library on NPM or Crates, but implementing a GCA is very straightforward.

Due to distinguishability, GCAs potentially leak some information about related, deeply-nested sibling nodes as the Hamming approaches zero. Our namefilter GCAs have a cardinality of 47 by default, which is much deeper than most file paths.

We considered using XOR or Cuckoo filters instead of class Bloom filters. XOR is very close to the theoretic efficiency limit, but is very new and the library untested. Cuckoo filters would provide around an additional 4 path segments with the same false-positive rate, but we lose the single-bit-collision of Bloom filters which is actually an advantage for obsfucation.

## Storage Tree

MMPT

Unlike the public file system, the private file system is stored as a tree. More specifically, this is a SHA256-based [Modified Merkle Patricia Tree \(MMPT\)](https://eth.wiki/en/fundamentals/patricia-tree).  




At the IPFS layer for the private section is filled exclusively with “locked“ / virtual secret nodes \(“SNode”s\). They have 2048-bit names, encoded in [base64URL](https://datatracker.ietf.org/doc/html/rfc4648#section-5), for a 32-character UTF8 name.



 hased to 256-bits, arranged in an append-only SHA256 [Modified Merkle Patricia Tree \(MMPT\)](https://eth.wiki/en/fundamentals/patricia-tree). The names are deterministic, and collisions extrememly unlikely in the resulting 2^256 \(~1.16 x 10^77\) namespace. More detail on the naming system is available in its own section.

The MPT layout allows for efficient validation that an update is append-only \(and thus nondestructive\).

## ENode Content

A vnode that has been secured in this way is called an ”encrypted virtual node”. The contents of these nodes is largely the same as their plaintext counterparts, plus a key table for their children.

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

