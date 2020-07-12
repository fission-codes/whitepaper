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
data VirtualNode
  = DirectoryNode Directory
  | FileNode      File
  | Symlink       DNSLink
  
data File = File
  { metadata   :: Metadata
  , rawContent :: CID
  }
  
data Directory = Directory
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
  , ...and so on
  }

```

## Protocol Layer

```haskell
data Node = Node
  { metadata :: Metadata
  , 
  }

data File = File
  { metadata   :: Metadata
  , rawContent :: CID
  }
  
data Directory = Directory
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
  , ...and so on
  }
```



## 

