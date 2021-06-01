---
description: Globally visible and addressable cleartext
---

# Public

The public directory contains regular, unencrypted, structured data. This includes previous versions, metadata, symlinks, and so on. This lives in the top-level `/public` directory. All of the content is publicly viewable, including previous versions.

## Platform Layer

At the platform layer, a public virtual node has the following shape:

```haskell
data VirtualNode -- could be paramaterized over protocol type later
  = DirectoryNode Directory
  | FileNode      File 
  | Symlink       DNSLink
  | MovedNode     Path
  | RawProtocol   IPFSNode
  
data File = File
  { metadata   :: Metadata
  , rawContent :: CID
  }
  
data Directory protocol = Directory
  { metadata :: Metadata
  , index    :: Map Text VirtualNode
  }
  
data Metadata = Metadata
  { history     :: Maybe History
  , unixMeta    :: UnixMeta
  , wnfsVersion :: SemVer
  }
  
data History = History
  { previous :: VirtualNode
  , event    :: Event
  }
  
data UnixMeta = UnixMeta
  { mtime :: UTCTime
  , ctime :: UTCTime
  , mode  :: UnixFileMode
  , type_ :: UnixNodeType
  }

data IPFSNode = IPFSNode
  { links :: [(Text, RawProtocol)]
  , data_ :: Maybe ByteString
  }
  
data IPFSLink = IPFSLink
  { name :: Text
  , hash :: CID
  , size :: Quantity Bytes
  }
```

## Data Layer

The data layer strips out much of the above structure, boiling it down to a series of `IPFSLink`s. This is fundamentally achieved by a function:

```haskell
serializeForProtcol :: VirtualNode -> IPFSNode
```

In this function, much of the metadata is compacted into CBOR files, for efficiency and convenience with the IPFS-supported `dag-cbor`.

Here is an intermediate abstraction to help describe the layout:

```haskell
data IPFSSerialized = IPFSSerialized
  { metadata :: CBOR
  , dagCache :: CBOR
  , previous :: CID
  , userland :: Userland
  }
  
data Userland = Either [(Text, IPFSLink)] IPFSNode
```

Note that links are NOT flattened into a single node. WNFS maintains a sepacial separate namespace for userland. This is a 2-layer approach:

```text
   +———————————————————+
   |                   |
   |     IPFSNode      |
   |                   |
   |     +———————+     |
   |     | Links |     |
   |     +———————+     |
   |     / |   | \     |
   +————/——|———|——\————+
       /   |   |   \
    Prev   |   |   Userland
     /     |   |      \
    / dag.cbor |       \ 
<——*       | meta.cbor  \
           |   |         \
       +————+ +————+ +———————————————+
       |DATA| |DATA| |               |
       +————+ +————+ |   IPFSNode    |
                     |               |
                     |   +———————+   |
                     |   | Links | <———— the directory index
                     |   +———————+   |
                     |    /  |  \    |
                     +———/———|———\———+
                        /    |    \
                      ...   ...   ...
```

{% hint style="warning" %}
Note that the prev link SHOULD be reified in a protocol link rather than in the cache to ensure that the link is real, the file will never be dropped \(if the root user breaks a layer\), and make it faster to verify.
{% endhint %}

## Write Access

Write access may be granted via UCAN. In this case, the platform-layer \(pretty\) path to the node is updatable arbitrarily, as are its nested contents. However, this necessitates updating the links in the Merkle structure above, as well as portions of metadata \(such as size of contents\). This is a rote mechanical procedure, and will be checked by the verifier.

{% hint style="warning" %}
It bears repeating that while this does create updated parent nodes, it wil lbe handled mechanically by the WNFS client. The verifier is able to easily and mechanically confirm these updates, and will reject them if submitted incorrectly.
{% endhint %}

