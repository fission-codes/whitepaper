# File Layer

The platform layer has decypted \(or ”unlocked”\) access to private vnodes.

### Unlocking

To read a node, the user needs to have the key either available from another node which they have access to, in the `shared_with_me` or `shared_by_me` sections, or stored on their system directly.

To read or ”unlock“ a private node, you need the node and its key:

```haskell
read :: AES256 -> SecretNode -> DecryptedNode
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
  , revision   :: SpiralRatchet
  , rawContent :: CID  -- can be split across many sections if we want to obscure files
  }

data PrivateDirectory = PrivateDirectory
  { metadata       :: Metadata
  , bareNameFilter :: BareNameFilter
  , revision       :: SpiralRatchet
  , previous       :: EncryptedLink
  , links          :: Map Text PrivateLink
  }

data PrivateLink = PrivateLink
  { name    :: Text -- Just the last segment in the path
  , key     :: AES256
  , pointer :: NameFilter -- Small overhead for consistency
  , size    :: Natural
  , mtime   :: UTCTime
  , isFile  :: Bool
  }
```

### Secure Recursive Read Access

The private section is recursively protected with AES-256 encryption. This is to say that each vnode is encrypted with an AES key, and each of its children are encrypted separately with their own randomly derived AES keys. A node holds the keys to each of its children. In this way, having a key for a node also grants read access to that entire subgraph.

### Revocation

Read access revocation is achieved by changing the AES key and linking to a higher node. As such, it is not recommended for a user with write permissions to rotate the key of the root of their subgraph, unless they’re able to redistribute that key somehow. For example, the root user is able to update the key for the root of the graph, and distribute that key to their other user instances by the `shared_by_me` mechanism.

{% hint style="danger" %}
A node with no valid key pointing at it is said to be orphaned, since it has no parents that are capable of acessing the data locked in the node. It may be marked for garbase collection by the root user.
{% endhint %}

### Decrypted Nodes

Since the structure of a cryptDAG is hidden completely from the outside world, there is a very strict separation between the platform layer, and how things are organized at the protocol layer. There are still two layers, but the protocol layer is more closely relied on by the platform layer.

The protocol layer describes encrypted nodes, with a special naming scheme and organized in a Merkle Patricia Tree \(more below in Storage Layout\). These can be converted to a decrypted virtual node via an _external_ symmetric key.

## Storage Layout

Encrypted virtual nodes are kept in a Merkle Patricia tree \(MPT\), organized by a blinded file name \(see more in the naming section below\).

The probabilistic nature of XOR filter filenames does mean that related files are more likely to be placed near each other in the MPT, while not giving away why they are placed in that part of the tree. Some direct descendants or siblings will be in far other parts of the tree, depending on the position of the first different bit. The filter is fixed-size, which further simplifies this layout.

This layout greatly improves write access verification time, while eliminating the plaintext tree structure. An authorized user reconstructs the human-readable DAG at runtime by following links in decrypted nodes. Their links point to files in the MPT \(or faster via the cache\).

## Move Markers

Rotated ratchet.

## Root Self-Storage

Store the namefilter and/or CID and key. Keep cache mapping all seen namefilter =&gt; keys?

