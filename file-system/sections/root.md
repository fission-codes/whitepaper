# Root

The WNFS root is a specialized structure that keeps the disparate segments together.

```haskell
data RootNode = RootNode
  { version    :: SemVer
  , p          :: BareIPFS -- Pretty tree
  , public     :: PublicNode
  , private    :: MMPT
  , privateLog :: CBOR -- Array of `sha3(cid)`s, newest-to-oldest
  , shared     :: Shared
  }
```

Since this layer is completely controlled by WebNative, we don't need to worry about metadata elements conflicting with userland.

* Pretty is repesented as simply `/p/` to be unobtrusive in URLs.

## Private Log

The private log is a cache of the changes to the MMPT. Since we're going to some lengths to hide the relationships between entries, versioning presents a challenge. With a simple pointer back to the previous version, correlating entries is fairly trivial. We want to hide these relationships while still allowing a quick way to validate the inclusion of another MMPT \(or conversely, if it is contained in the other\).

To achieve this, we maintain a log of hashed MMPT history. This is a newest-to-oldest array of the hashes of `private` CIDs. Each entry is: `sha3(current_cid)`. It is mutable \(i.e. we do not persist references to previous versions of the set\), and clearable \(you can drop old revisions if the array becomes too large\). It is recommended that you keep as many entries as will fit in one IPFS block \(around 950 including CBOR delimiters\).

It is very easy to construct a SHA of an MMPT CID that you currently have — you simply hash it. To reconstruct the order of MMPT versions \(and thus correlate items\), you would need to _already_ have the MMPTs.

### Cache Corruption

It is always possible to do a direct structural comparison \(hence this being a cache\), but comparing hashes is much faster. However, this does mean that if this cache becomes corrupt, it is possible to start again.

### Why Not a Bloom Filter?

A bloom filter would further obsfucate the set, and has very fast lookups. The problem is false positives. Even 1/10M is something that we will run into at scale, and this cache is primarily used to do head detection, which needs to be pretty robust.

