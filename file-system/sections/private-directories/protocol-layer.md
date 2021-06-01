# Protocol Structure

## Storage Layout

At the IPFS layer for the private section is filled exclusively with “locked“ / encrypted private virtual nodes \(“ENode”s\). They have 2048-bit \(256 byte\) names, encoded in base64URL, for a 32-character UTF8 name.



 hased to 256-bits, arranged in an append-only Modified Merkle Patricia tree \(MMPT\). The names are deterministic, and collisions extrememly unlikely in the resulting 2^256 \(~1.16 x 10^77\) namespace. More detail on the naming system is available in its own section.

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

## ENode Naming

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

