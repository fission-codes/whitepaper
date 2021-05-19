# Anatomy

WNFS is a Merkle DAG where the terminal nodes are _either_ empty directories or files. It is efficient in low-level operations \(e.g. deduplication, network sync\).

## Layers

The term "layer" refers exclusively to the stack of abstractions, with concrete or low-level constructs at the bottom, and progressively more abstract and ”human” as we go up. 

WNFS is built up from Merkle-linked structures, and needs to operate at several layers within the stack. WNFS is built on top of the [Interplanetary File System \(IPFS\)](https://ipfs.io/), but that may not always be the case. The core requirement is [content addressing](https://en.wikipedia.org/wiki/Content-addressable_storage). As such, many of the abstractions are slightly different from the raw IPFS ecosystem.

### Protocol Layer

This layer describes how we need to concretely represent our data to the network. This is a data layer to be consumed by whichever substrate WNFS is running on.

### Application Layer

The application layer is an abstraction over models. Having the full power of computation at runtime means that we can subordinate extraneous detail, and provide a familiar model and clean interface to end users.

## Section

WNFS has several well defined sections defined at the root of the DAG. These include \(but are not limited to\) the public, private, and shared sections. There is a strict separation between these sections, for many reasons, but importantly access control — both for users an a separation between userland and kernelspace.

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

An encrypted node is a virtual node \(or subtype\) which has been encrypted. An external key is required to read this node. See the section on the private tree for more detail of the architecture of this in practice.

{% tabs %}
{% tab title="TypeScript" %}
```typescript
read(key: AES256, eNode: Encrypted<VNode>): Result<Failure, VNode>
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
read :: AES256 -> Encrypted VirtualNode -> Either Failure VirtualNode
```
{% endtab %}
{% endtabs %}

### Node Segments

A node is broken into two segments: header and content. There are a number of reasons for this layout, not least of which is keeping the content \(userland\) in a strictly separated namespace from the header \(system managed\).

These segments are stored as separate nodes at the protocol layer, but together at the level of application abstraction.

```text
+---------------------------+
|        VirtualNode        |
|                           |
|  +--------+  +---------+  |
|  | Header |  | Content |  |
|  +--------+  +---------+  |
|                           |
+---------------------------+
```

#### Header

Contains information _about_ the node and its contents. This includes information such as node size, tags, caches, indexes, and pointers to previous versions. This segment _does_ cause changes in structure at the protocol layer with elements like previous version pointers.

The header is primarily system \(SDK\) managed, but may be influenced by the user \(e.g. adding tags\).

{% hint style="info" %}
The information stored in the header segment is _descriptive._ It is structural at the protocol layer, but not at the application layer.
{% endhint %}

#### Content

The actual information storage and linking to other nodes. Links to the actual raw contents of a file. This is an internal detail, and that this is a separate segment is generally hidden from end users.

{% hint style="info" %}
The information stored in the content segment is primarily _operational._ It contains the primary semantic links that get exposed to the end user.
{% endhint %}

## DAG Reduction

A copy of a larger structure, with some data removed. This process is inherently lossy by definition, and never introduces new information to the reduction.

### Example

`pretty` is a DAG reduction of the `public` section. It's a reduction index because it contains precisely the same files and paths, but with extra detail removed. This is held directly in the DAG to facilitate human readable URLs.

