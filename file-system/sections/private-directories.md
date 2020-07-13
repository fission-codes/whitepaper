# Private

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

