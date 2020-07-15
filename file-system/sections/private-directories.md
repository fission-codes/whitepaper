# Private

The private section is an extension of the functionality in the public section. The additional goals of the private tree are to maximize privacy, security, flexibility, and autonomy for users.

{% hint style="danger" %}
Write access in FLOOFS is a semi-trusted setup. The root user delegates write access to subgraph. FLOOFS has aimed to be correct-by-construction where possible. However, since service verifiers cannot validate the inner contents of a write beyond access to a \(hidden\) file path, the submitter/writer may fill that node with anything.

Validating the contents homomorphically or with some zero knowledge setup is technically possible, but the technology is still early. At minimum the efficiency need to improve greatly before that’s viable on deep DAG writes on a low-powered smartphone.
{% endhint %}

## Protocol Layer

### Storage Layout

The protocol layer for the private section is filled exclusively with “locked“ /encrypted private virtual nodes \(“ENode”s\). They have 512-bit \(64 byte\) names arranged in a Merkle Patricia tree \(MPT\). The names are deterministic, and collisions extrememly unlikely in the 2^512 \(10^77\) namespace. More detail on the naming system is available in its own section.

The MPT layout allows for efficient validation that an update is append-only and thus nondestructive.

### ENode Content

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

### ENode Naming

Files names are a consistent legth: 512-bits. They are XOR filters, and have a consistent number of bits flipped \(e.g. 320\). For instance \(expressed in base58\):

```text
yP4cqy7jmaRDzC2bmcGNZkuQb3VdftMk6YH7ynQ2Qw4zktKsyA9fk52xghNQNAdkpF9iFmFkKh2bNVG4kDWhsok
```

The method of generating these names is explained in the decrypted node section.

### Hash Table Cache

{% hint style="info" %}
It’s concievable that this structure could be implemented ”natively” in IPFS by using the `Links` array, but would require investigating the bounds and performance characteristics of the protocol itself.

Another, option would be to store these as records in a SQLite database written to FLOOFS. This will probaby has better performance characteristics, even if it only has one table. It will also likely be able to do very efficient substring matching on keys \(file names\). Also:

> SQLite Is Public Domain

If SQLite works for our use case, we can ignore the hashing and whatnot below to better facilitate lookups by string match.
{% endhint %}

Currently traversing an MPT over the network of \(max\) depth 512 can take a while. Following paths in a decrypted node \(see below\) requires doing a large number of lookups, and having an in-memory structure to do this with speeds things up significantly.

A totally flat, append-only namespace with unique names has a number of nice properties. For one, it can be represented as a simple sorted array, which we do here.

{% hint style="warning" %}
Please note that this cache is just that: a cache. It can be destroyed and reconstituded at any time \(for example if it were to become corrupted\). That said, given that it’s publicly viewable, a validator can check that it conforms to the updates in the MPT.
{% endhint %}

The file namespace is larger than we could ever use. It includes many redundant bits to aid in oblivious access control \(more below\), but need not exist in this lookup table. Instead we use a SHA-256 to reduce the size by nearly half in the hash table. Between the hash and CID, each record occupies ~64 bytes \(depending on generation of CID\). This table can store ~15k entries / MB. Most of the underlying blocks will stay the same size, so syncing updates is very efficient in the normal case.

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

## Application Layer

The application layer has decypted \(or ”unlocked”\) access to private vnodes.

### Unlocking

 To read a node, the user needs to have the key either available from another noe which they have access to, in the `shared_with_me` or `shared_by_me` sections, or stored on their system directly.

To read or ”unlock“ private node, you need the node and its key:

```haskell
read :: AES256 -> EncryptedNode -> DecryptedNode
```

### Unlocked Private Node Schema

```haskell
data DecryptedNode
  = DecryptedDirectory PrivateDirectory
  | DecryptedFile      PrivateFile
  | DecryptedSymlink   Symlink
  | DecryptedMovedTo   PathFilter -- Must maintain the same key to work

data PrivateFile = PrivateFile
  { metadata   :: Metadata
  , key        :: AES256
  , rawContent :: CID  -- can be split across many sections if we want to obscure files
  }

data PrivateDirectory = PrivateDirectory
  { metadata     :: Metadata
  , parentFilter :: MinimalFilter
  , revision     :: Natural -- Version counter for this exact AES key
  , previous     :: EncryptedLink
  , children     :: Map Text PrivateLink
  , dagCache     :: DAGCache
  }

data DAGCache 
  = Leaf CID
  | Branch (Map TextPath DAGCache)
  
data PrivateLink = PrivateLink
  { path    :: Text -- Just the last segment in the chain
  , key     :: AES256
  , pointer :: CID
  }
```

### Secure Recursive Read Access

The private section is recursively protected with AES-256 encryption. This is to say that each vnode is encrypted with an AES key, and each of its children are encrypted separately with their own randomly derived AES keys. A node holds the keys to each of its children. In this way, having a key for a node also grants read access to that entire subgraph.

### Revocation

Read access revocation is achieved by changing the AES key and linking to a higher node. As such, it is not recommended for a user with write permissions to rotate the key of the root of ther subgraph, unless they’re able to redistribute that key somehow. For example, the root user is able to update the key for the root of the graph, and distributet that key to their other user instances by the `shared_by_me`mechanism.

{% hint style="danger" %}
A node with no valid key pointing at it is said to be orphaned, since it has no parents that are capable of acessing the data locked in the node. It may be marked for garbase collection by the root user.
{% endhint %}

### Decrypted Nodes

Since the structure of a cryptDAG is hidden completely from the outside world, there is a very strict separation between the application layer, and how things are organized at the protocol layer. There are still two layers, but the protocol layer is more closely relied on by the application layer.

The protocol layer describes encrypted nodes, with a special naming scheme and organized in a Merkle Patricia Tree \(more below in Storage Layout\). These can be converted to a decrypted virtual node via an _external_ symmetric key.

## Storage Layout

Encrypted virtual nodes are kept in a Merkle Patricia tree \(MPT\), organized by a blinded file name \(see more in the naming section below\).

The probabilistic nature of XOR filter filenames does mean that related files are more likely to be placed near each other in the MPT, while not giving away why they are placed in that part of the tree. Some direct descendants or siblings will be in far other parts of the tree, depending on the position of the first different bit. The filter is fixed-size, which further simplifies this layout.

This layout greatly improves write access verification time, while eliminating the plaintext tree structure. An authorized user reconstructs the human-readable DAG at runtime by following links in decrypted nodes. Their links point to files in the MPT \(or faster via the cache\). The low-level flow is always pointing back to the table.

A sequence diagram isn’t _perfect_ for this use case, but gets the job done. Here’s an example flow to the next node:

```text
Raw CIDs    Hash Table      Node A   Locked[Node B]      Node B
    |            |             |          |                |
    |            | hash(foo@N)?|          |                |
    |            |<————————————|          |                |
    |            |             |          |                |
    |            |————————————>|          |                |
    |            |  Qm124356!  |          |                |
    |            |             |          |                |
    |   Qm12345? |             |          |                |
    |<—————————————————————————|          |                |
    |            |             |          |                |
    |————————————————————————————————————>|                |
    | Raw Chunks |             |          |                |
    |            |             |  AES256  |                |
    |            |             |—————————>|                |
    |            |             |          |    Decrypt     |
    |            |             |          |———————————————>|
    |            |             |          |                |
```

## Read Access

FLOOFS has a recursive read access model known as a cryptree \(technically a cryptDAG in our case\). Each Decrypted Virtual Node contains the keys to its children nodes. It also includes the human-readable path name, and the of the revision that it’s aware of \(more below\).

### Deterministic Seek Ahead

### Secret Names

A lot can be gleaned from a node’s name, or a tree structure.

Fully zero knowledge methods do exist for this, but are quite new and do not \(yet\) perform fast enough to be practical for this use case. FLOOFS opts to hide as much information as possible while remaining reasable on a low-end smartphone.

### Path Filters

To facilitate a determinist-but-highly-obsfucated naming scheme, as well as give a verifier that doesn’t have read access the ability to check that the writer can submit updates to that path.

Ths is achieved with XOR Filters \(a more efficient Bloom Filter\). These structures allow validation that an element is in some compressed/obfuscated set.

```text
child_filter = hash_reduction(
      parent_filter
  AND hash(aes_key) 
  AND hash(revision ++ aes_key)
)
```

> A previous design used the file path plus a nonce and its index. If the path changed, this would break any write certificates. The current design requires that you only know only the AES key used to encrypt a node \(we assume read access as a prerequisite to write  access\). The latest revision number can be found by scanning the store \(more on this later\), but is also stored on in node.

#### Recursive Hash Accumulation

With the naive approach, it is easy to tell which nodes are higher in the DAG: ther filters are sparser. To further obsfucate the path without invalidating write tokens, we use a deterministic method to produce a consistently-sized filter \(to some depth\) via hash reduction.

Hash acculumation works by recursively `AND`ing the hash of the previous filter with the current filter, to some depth. As a simple example:

```yaml
parent_base:   1100100101011001

key:           “abcdef”
hash(key):     0000000110000000
hash(version): 0000000100000001

base:          1100100101011001
               0000000110000000
           AND 0000000100000001
           ====================
               1100100111011001
               
hash(base):    0100001000000010

base+1:        1100100111011001
           AND 0100001000000010
           ====================
               1100101111011011
               
hash(base+1):  0010000010001000

base+1+2:      1100101111011011
           AND 0010000010001000
           ====================
               1110101111011011
```

```haskell
-- PSEUDOCODE

toEntry = bloom . sha256

hashReduction parent aesKey version =
  constantSized aesKey (parent .&. toEntry (aesKey <> show version))
  
constantSized aesKey bits = 
  if countOnes == 320
     then bits
     else constantSize (bits .&. toEntry (awsKey <> show bits))
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











Private files and directories are encrypted symmetrically.

We specify a format for an encrypted file tree, inspired by Cryptrees as described by [Grolimund et al](https://ieeexplore.ieee.org/document/4032481).

Our Cryptree is a recursive data structure where each "folder" or node in the tree contains:

* the name of a symmetric encryption algorithm
* a symmetric key
* an array of links to other nodes, each of which is encrypted by the symmetric key.

These nodes form a Directed Acyclic Graph \(DAG\), similar in form to an IPFS merkle-DAG.

These nodes are described as such:

```typescript
type EncryptedDAGNode = {
  alg: string // symmetric encryption algorithm
  key: Buffer // symmetric key
  links: Link[]
}

type Link = {
  name: string
  cid: CID // IPFS-compatible multihash (CIDv0/CIDv1)
  size?: number // the size of all children
}
```

In the context of the node, The key is stored as plain bytes. Therefore, any time an `EncryptedNode` is uploaded to the IPFS network, it must be encrypted itself.

DAG nodes should be CBOR-encoded prior to being encrypted.

This forms a structure such that access to a given node allow access to any of its children, but none of its parents.

Any node in the DAG can be shared by sharing the CID of the node as well as the key to decrypt it.

The private directory of the Fission File System uses AES keys with the `AES-CTR` algorithm.

