---
description: Globally visible and addressable plaintext
---

# Public

The public directory contains regular, unencrypted, structured data. Each VNode contains more This includes previous versions, metadata, symlinks, and so on. This lives in the top-level `/public` directory. All of the content is publicly viewable, including previous versions.

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
  , dagCache :: JSON
  }
  
data Metadata = Metadata
  { history       :: Maybe History
  , unixMeta      :: UnixMeta
  , floofsVersion :: SemVer
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

## Protocol Layer

The protocol layer strips out muh of the above structure, and reduces it to as eries of `IPFSLink`s. This is fundamentally acheived by a function:

```haskell
serializeForProtcol :: VirtualNode -> IPFSNode
```

In this function, much of the metadata is compacted into CBOR files \(for efficiency\).

Here is an intermedate abstraction to help describe the layout:

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

## DAG Cache

The protocol layer is the source of truth for linked data. However, to improve performance, FLOOFS keeps a \(recursive\) cache of the entire sub-DAG in the protocol layer. We are told that this optimization is being worked on at the protocol layer, but this is our performance optimization in the meantime. We name this cache `dag.cbor`

The insight is that describing even a very large DAG in JSON or CBOR is more efficient over the network than is following a series of links in ”pass the bucket” linear traversal \(where each iteration may be a network request\).

### Trees Not DAGs

Th purpose of the cache is to help improve perfomance of deep link lookups. While the structure is fundamentally a DAG, the encoding of non-trees is less streamlined in CBOR or JSON. There are very simple path methods available for there formats if they’re represented directly as trees. Theo ther opion requires much jumping around lookup tables, rather than simply following a nested path.

This is a minor space/time tradeoff: inthe cache all DAGs are converted to trees, with duplication in place of shared pointers. Since the structure is Merkelized, we don’t run the risk of mistaking disparate nodes for each other, and representing the structure as a tree is equally correct \(if more verbose\).

### Schema

The cache describes the platform layer, not the protocol layer. This means that it is not constrained by the needs of the Merkle DAG.

```haskell
data CacheNode
  = CacheFile CID
  | Directory CacheDirectory
  | Symlink   DNSLink

data CacheDirectory = CacheDiectory
  { cid      :: CID
  , contents :: Map Text CacheNode
  }
```

### Interaction with Versioning

This cache records a single generation only. It does not include references to previous versions. Temporal operations aways occur on the protocol-level DAG, or abstractly accessed through the WNFS platform layer. The DAG cache should be kept as thin as possible, as this may become quite large. Users do not expect history to be a  lightening fast operation. It is still accessible by looking at the concrete \(uncached\) virtual node.

### FAQ

Why not only keep this cache at the file system root? Deep linking performance is greatly improved by being able to pull a single file off the network, and inspecting it locally.

## Write Access

Write access may be granted via UCAN. In this case, the platform-layer \(pretty\) path to the node is updatable arbitraily, as are its nested contents. However, this necessitates updating the links in the merkle structure above, as well as portions of metadata \(such as size of contents\). This is a rote mechanical procedure, and will be checked by the verifier.

{% hint style="warning" %}
It bears repeating that while this does create updated parent nodes, it wil lbe handled mecanically by the FLOOFS client. The verifier is able to easily and mechanically confirm these updates, and will reject them if submitted incorrectly.
{% endhint %}

