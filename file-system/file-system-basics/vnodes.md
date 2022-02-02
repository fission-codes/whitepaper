# Virtual Nodes

A virtual node ("vnode") is an abstraction over files and directories. It describes some basic structure that all nodes in the graph conform to:

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

## Encrypted Node

An encrypted node is a virtual node (or subtype) which has been encrypted. An external key is required to read this node. See the section on the private tree for more detail of the architecture of this in practice.

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

## Node Segments

A node is broken into two segments: header and content. There are a number of reasons for this layout, not least of which is keeping the content (userland) in a strictly separated namespace from the header (system managed).

These segments are stored as separate nodes at the protocol layer, but together at the level of application abstraction.

```
+---------------------------+
|        VirtualNode        |
|                           |
|  +--------+  +---------+  |
|  | Header |  | Content |  |
|  +--------+  +---------+  |
|                           |
+---------------------------+
```

### Header

Contains information _about_ the node and its contents. This includes information such as node size, tags, caches, indexes, and pointers to previous versions. This segment _does_ cause changes in structure at the protocol layer with elements like previous version pointers.

The header is primarily system (SDK) managed, but may be influenced by the user (e.g. adding tags).

{% hint style="info" %}
The information stored in the header segment is _descriptive._ It is structural at the protocol layer, but not at the application layer.
{% endhint %}

### Content

The actual information storage and linking to other nodes. Links to the actual raw contents of a file. This is an internal detail, and that this is a separate segment is generally hidden from end users.

{% hint style="info" %}
The information stored in the content segment is primarily _operational._ It contains the primary semantic links that get exposed to the end user.
{% endhint %}
