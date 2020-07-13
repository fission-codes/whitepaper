# Private

The private section is an extension of the functionality in the public section. The additional goals of the private tree are to maximize privacy, security, flexibility, and autonomy for users.

{% hint style="danger" %}
Write access in FLOOFS is a semi-trusted setup. The root user delegates write access to subgraph. FLOOFS has aimed to be correct-by-construction where possible. However, since service verifiers cannot validate the inner contents of a write beyond access to a \(hidden\) file path, the submitter/writer may fill that node with anything.

Validating the contents homomorphically or with some zero knowledge setup is technically possible, but the technology is still early. At minimum the efficiency need to improve greatly before that’s viable on deep DAG writes on a low-powered smartphone.
{% endhint %}

* MOVE POINTER AS A KIND OF NODE!
* Runtime binary search lazy lookup

## Secure Recursive Read Access

The private section is recursively protected with AES-256 encryption. This is to say that each vnode is encrypted with an AES key, and each of its children are encrypted separately wit their own randomly derived AES keys. A node holds the keys to each of its children. In this way, having a key for a node also grants read access to that entire subgraph.

## Encrypted Virtual Nodes “ENodes”

A vnode that has been secured in this way is called an ”encrypted virtual node”. The contents of these nodes is largely the same as their plaintext counterparts, plus a key table for their children.

The core difference is the encrypted storage \(protocol layer\), and secrecy of the key used to start the decryption process. The key is always external to the ENode, and its not aware of whch key was used to create it.

### Encrypted Node Schemata

```haskell
data EncryptedNode = EncryptedNode CID -- simple!

read :: AES256 -> EncryptedNode -> DecryptedNode

data DecryptedNode
  = DDirectory DecryptedDirectory
  | DFile      File
  | DSymlink   Symlink
  | DMoved     
  | DKeyRotation

data DecryptedDirectory = DecryptedDirectory
  { metadata :: Metadata
  , previous :: EncryptedLink
  , revision :: Natural 
  , children :: Map Text EncryptedLink
  }
  
data EncryptedLink = EncryptedLink
  { key     :: AES256
  , pointer :: EncryptedNode
  }
```



-—-

—

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

