# Root

The WNFS root is a specialized structure that keeps the disparate segments together.

```haskell
data RootNode = RootNode
  { version    :: SemVer
  , p          :: BareIPFS -- Pretty tree
  , public     :: PublicNode
  , private    :: MMPT
  , privateLog :: CBOR -- Set of `sha3(cid)`s
  , shared     :: Shared
  }
```

Since this layer is completely controlled by WebNative, we don't need to worry about metadata elements conflicting with userland.

* Pretty is repesented as simply `/p/` to be unobtrusive in URLs.

## Private Log

The private log is a cache of the changes to the MMPT. Since we're going to some lengths to hide the relationships between entries, versioning presents a challenge. With a simple pointer back to the previous version, correlating entries is fairly trivial. We want to hide these relationships while still allowing a quick way to validate the inclusion of another MMPT \(or conversely, if it is contained in the other\).

To achieve this, we maintain a log of hashed MMPT history. This is a set \(i.e. alphanumerically sorted CBOR array\) of the hashes of `private` CIDs. To be clear, each entry is: `sha3(current_cid)`. It is mutable \(i.e. we do not persist references to previous versions of the set\).

It is always possible to do a direct structural comparison \(hence this being a cache\), but comparing hashes is much faster. Howevre, this does mean that if this cache becomes corrupt, it is possible to start again.

