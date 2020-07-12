---
description: Globally visible and addressable plaintext
---

# Public Directory

The public directory contains regular, unencrypted, structured data. Each VNode contains more This includes previous versions, metadata, symlinks, and so on. This lives in the top-level `/public` directory. All of the content is publicly viewable, including previous versions.

## 

As mentioned in Anatomy, virtual nodes are an abstraction over files and 

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
           NodeRoot
          /    |   \ 
         /     |    \
    Previous   |   UserLand
       /   meta.cbor  |  \
      /               |   \
<————*                V    V
```

