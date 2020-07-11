# Concepts

FLOOFS is a DAG where the terminal nodes are _either_ empty directories or files. It is also a Merkle DAG to aid in low level operations \(e.g. deduplication, sync\), and to increase the performance of write access control.

## Layer

The term "layer" refers exclusively to the stack. FLOOFS is built up from Merkle linked structures, and needs to operate at several layers within the stack. At time of writing, FLOOFS runs on top of IPFS, but that may not always be the case. As such, we use slightly different abstractions from that project.

### Protocol Data Layer

This layer describes how we need to concretely represent our data to the network. This is a data layer to be consumed by whichever substrate FLOOFS is running on.

### Application Schema Layer

The application layer is an abstraction over models. Having the full power of computation at runtime means that we can subordinate extraneous detail, and provide a familiar model and clean interface to end users.

## Section

FLOOFS has several well defined sections defined at the root of the DAG. These include \(but are not limited to\) the public, private, and shared sections.

## Node 

### Virtual Node

A virtual node \("vnode"\) is an abstraction over files and directories. It describes some basic structure that all nodes in the graph conform to:

{% tabs %}
{% tab title="TypeScript" %}
```typescript
type VirtualNode = File | Directory | Symlink
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
data VirtualNode
  = FileNode      File
  | DirectoryNode Directory
  | Symlink       DNSLink
```
{% endtab %}
{% endtabs %}

### Encrypted Node

An encrypted node is a virtual node \(or subtype\) which has been encrypted. An external key is required to read this node.

{% tabs %}
{% tab title="TypeScript" %}
```typescript
read(eNode: Encrypted<VNode>, key: AES256): Result<Failure, VNode>
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
read :: Encrypted VirtualNode -> AES256 -> Either Failure VirtualNode
```
{% endtab %}
{% endtabs %}

### Node Contents

### Raw

The actual raw contents of a file. This is an internal detail, and generally hidden from end users. It is useful for implementers, however.

## Reduction Index

A copy of a larger structure, with some data removed. This process is inherently lossy by definition, and never introduces new information to the reduction.

### Example

The `pretty` section is a reduction of the `public` section. It's a reduction index because it contains precisely the same files and paths, but with extra detail removed. This is held directly in the DAG to facilitate human readable URLs.

