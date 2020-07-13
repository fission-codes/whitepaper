# Private

The private section is an extension of the functionality in the public section. The additional goals of the private tree are to maximize privacy, security, flexibility, and autonomy for users.

{% hint style="danger" %}
Write access in FLOOFS is a semi-trusted setup. The root user delegates write access to subgraph. FLOOFS has aimed to be correct-by-construction where possible. However, since service verifiers cannot validate the inner contents of a write beyond access to a \(hidden\) file path, the submitter/writer may fill that node with anything.

Validating the contents homomorphically or with some zero knowledge setup is technically possible, but the technology is still early. At minimum the efficiency need to improve greatly before that’s viable on deep DAG writes on a low-powered smartphone.
{% endhint %}

* MOVE POINTER AS A KIND OF NODE!
* Runtime  \*lazy\* binary search lookup \(eventual progress\)
* Global counter

## Secure Recursive Read Access

The private section is recursively protected with AES-256 encryption. This is to say that each vnode is encrypted with an AES key, and each of its children are encrypted separately wit their own randomly derived AES keys. A node holds the keys to each of its children. In this way, having a key for a node also grants read access to that entire subgraph.

## Encrypted Virtual Nodes “ENodes”

A vnode that has been secured in this way is called an ”encrypted virtual node”. The contents of these nodes is largely the same as their plaintext counterparts, plus a key table for their children.

The core difference is the encrypted storage \(protocol layer\), and secrecy of the key used to start the decryption process. The key is always external to the ENode, and its not aware of whch key was used to create it.

### Encrypted Node Schemata \(Application Layer\)

```haskell
data EncryptedNode = EncryptedNode CID -- simple!

read :: AES256 -> EncryptedNode -> DecryptedNode

data ETree 
  = ETreeNode 
  | ETreeLeaf

data ETreeLeaf 
  = ETreeLeaf CID

data ETreeNode = ETreeNode
  { zero :: (Binary, ETree) -- NOTEThese are *sides*, and may terminate directly
  , one  :: (Binary, ETree)
  }  

data DecryptedNode
  = DDirectory DecryptedDirectory
  | DFile      DecryptedFile
  | DSymlink   DecryptedSymlink
  | DMovedTo   Path -- Must maintain the same key

data DecryptedDirectory = DecryptedDirectory
  { metadata :: Metadata
  , previous :: EncryptedLink
  , revision :: Natural -- Counter for this exact path
  , children :: Map Text EncryptedLink
  }
  
data EncryptedLink = EncryptedLink
  { key     :: AES256
  , pointer :: EncryptedNode
  }
```

### Decrypted Nodes

Since the structure of a cryptDAG is hidden completely from the outside world, there is a very strict separation between the application layer, and how things are organized at the protocol layer. There are still two layers, but the protocol layer is more closely relied on by the application layer.

The protocol layer describes encrypted nodes, with a special naming scheme and organized in a Merkle Patricia Tree \(more below in Storage Layout\). These can be converted to a decrypted virtual node via an _external_ symmetric key.

## Storage Layout

Encrypted virtual nodes are kept in a Merkle Patricia tree \(MPT\), organized by a blinded file name \(see more in the naming section below\).

The probabilistic nature of XOR filter filenames does mean that related files are more likely to be placed near each other in the MPT, while not giving away why they are placed in that part of the tree. Some direct descendants or siblings will be in far other parts of the tree, depending on the position of the first different bit. The filter is fixed-size, which further simplifies this layout.

This layout greatly improves write access verification time, while eliminating the plaintext tree structure. An authorized user reconstructs the semantically relevant DAG at runtime by following links in decrypted nodes. The links point to entries in the MPT, giving `O(1) ~ o(log n | n < 10)` access \(where `n` is the number of bits, which is constant in an XOR filter\). Organizing as a BST/Patricia tree is a very common approach for implementing hash tables like this one.

## Read Access

FLOOFS has a recursive read access model known as a cryptree \(technically a cryptDAG in our case\). Each Decrypted Virtual Node contains the keys 



```text
+—————————————+
| 
|   EncryptedNode 
|
|   +———————————————————+
|   |                   |
|   |   DecryptedNode   |
```



## Node Naming

A lot can be gleaned from a node’s name, or a tree structure.

Fully zero knowledge methods do exist for this, but are quite new and do not \(yet\) perform fast enough to be practical for this use case. FLOOFS opts to hide as much information as possible while remaining reasable on a low-end smartphone.

### Probabilistic Filter Names

To facilitate a determinist-but-obsfucated naming scheme, as well as give a verifier that doesn’t have read access the ability to check that the writer can submit updates to that path.

Ths is achieved with XOR Filters \(a more efficient Bloom Filter\). These structures allow validation that an element is in some compressed/obfuscated set.

```text
child_filter = hash_reduction(
      parent_base
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

