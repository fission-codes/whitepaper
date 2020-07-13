---
description: Globally visible and addressable plaintext
---

# Public

The public directory contains regular, unencrypted, structured data. Each VNode contains more This includes previous versions, metadata, symlinks, and so on. This lives in the top-level `/public` directory. All of the content is publicly viewable, including previous versions.

## Application Layer

At the application layer, a public virtual node has the following shape:

```haskell
data VirtualNode -- could be paramaterized over protocol type later
  = DirectoryNode Directory
  | FileNode      File
  | Symlink       DNSLink
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
  , previous :: CID
  , userland :: Userland
  }
  
data Userland = Either [(Text, IPFSLink)] IPFSNode
```

Note that links are NOT flattened into a single node. FLOOFS maintains a sepacial separate namespace for userland. This is a 2-layer approach:

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



### Schema

The cache describes the applicaion layer, not the protocol layer. This means that it is not constrained by the needs of the Merkle DAG.

```haskell
data 
```

### Interaction with Versioning

This cache records a single generation only. It does not include references to previous versions. Temporal operations aways occur on the protocol-level DAG, or abstractly accessed through the FLOOFS application layer. The DAG cache should be kept as thin as possible, as this may become quite large. Users do not expect history to be a  lightening fast operation. It is still accessible by looking at the concrete \(uncached\) virtual node.

### FAQ

Why not only keep this cache at the file system root? Deep linking performance is greatly improved by being able to pull a single file off the network, and inspecting it locally.

## Write Access



