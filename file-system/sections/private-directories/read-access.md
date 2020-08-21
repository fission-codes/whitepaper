# Read Access

WNFS has a recursive read access model known as a cryptree \(technically a cryptgraph in our case\). Each Decrypted Virtual Node contains the keys to its children nodes. It also includes the human-readable path name, and the of the revision that it’s aware of \(more below\).

## Deterministic Seek Ahead

Since name filters are deterministic, we can look up a version in constant time from the node cache. Below we go into greater detail about how progress is ensured, but is relevant to lookup.

If you have a pointer to a particular file, there is no way of knowing that you have been linked to the latest version of a node. The information that you do have includes everything that you need to construct a name filter.

* The current node’s AES key
  * Needed to decrypt this node
* The revision number of the current node
  * Stored on the node
* This node‘s bare name filter
  * Stored on the node

The user must always ”look ahead” to see if there have been updates to the file since they last looked. The three most common scenarios are that:  


1. No changes have been made
2. There have been been a small number of changes
3. There have substanial changes since

To balance these scenarios, we progressivley check for files at revision `r + 2^n` , where `r` is the current revision, and `n` is the search index. First we check the next revision. If it does not exist, we know that we have the latest version. If it does exist, check `r + 2`, then `r+4`, `r+8` and so on. Once there’s a missing version, perform a binary search. For example, if looking at a node at revision 42 that has been updated 123 times since your last recorded pointer, it takes 14 checks \(roughly `O(2 * log n)`\) to find the latest revision.

| Revision Number | Exists |
| :--- | :--- |
| 42 + 1 = 43 | Yes |
| 42 + 2 = 44 | Yes |
| 42 + 4 = 46 | Yes |
| 42 + 8 = 50 | Yes |
| 42 + 16 = 58 | Yes |
| 42 + 32 = 74 | Yes |
| 42 + 64 = 106 | Yes |
| 42 + 128 = 170 | No — First overshot! We now have an upper bound |
| 42 + 96 = 138 | Yes |
| 42 + 112 = 154 | Yes |
| 42 + 120 = 162 | Yes |
| 42 + 124 = 166 | No |
| 42 + 122 = 164 | Yes |
| 42 + 123 = 165 | Yes — No more search space, so found! |

## Lazy Progress

Anyone that can update a pointer can make permanent revision progress for themselves \(in localStorage or as a symlink in their FS\), or others if they have write access to this file system.

As the user traverses the private section \(down the Y-axis, across the X-axis\), they attempt to make progress in time \(forwards in the Z-axis\). If they find a node that’s ahead of a link, it updates that one link in memory. At the end of their search in 3-dimensions \(with potentially multiple updates\), they write the new paths to WNFS.

Note that they only need to do this with the paths that they actually follow! Progress in revision history does not need to be in lock step, and will converge over time.

Not all users with write access have the ability to write to the entire DAG. Writing to a subgraph is actually completely fine. Each traversal down a path will reach the most recently written node. The search space for that node is always smaller than its previous revisions, and can be further updated with oter links or newer child nodes.

This contributes back collaboratively to the overall performance of the system for all usres. If a malicious user writes a bad node, they can be overwritten with a newer revision by a user with equal or higher priviledges. Nothing is ever lost in WNFS, so reconstructing all links in a file system from scratch is _possible_ \(though compute intensive\).

## Secret Names

Structural information can be analyzed probabilistically from a node’s name, or a tree structure. Ergo we go to some effort to hide this information from users that should not know this information.

Fully zero knowledge methods do exist for this, but are quite new and do not \(yet\) perform fast enough to be practical for this use case. WNFS opts to hide as much information as possible while remaining reasable on a low-end smartphone.

## Name Filters

To facilitate a deterministic-but-highly-obsfucated naming scheme, as well as give a verifier that doesn’t have read access the ability to check that the writer can submit updates to that path.

This is achieved with Bloom Filters. These structures allow validation that an element is in some compressed/obfuscated set.

### Bare Name Filters

A bare name filter includes the smallest amount of information. It can be revealed to a a user that has read access to child nodes to help them build their own filters. They can be constructed by adding the current AES key and all of the keys above this node into an XOR filter. Since this is associative, equipping the node with its  own name filter is sufficient.

In WNFS, bare filters are generated for child nodes by adding the hash of the AES key to the existing filter, and storing that directly on the child for its further use. Bare filters are used in constructing names in the MPT, in the node cache, as well as by UCANs to authorize writing new nodes to a to a subgraph.

Bare filters do not include the version number, or fill out the filter to a particular level. We can see each filter entry as a path segment. The need not be ordered since each is entry is unique and randomly generated.

### Saturated Name Filters

To futher obscure the information in a name filter, we want to deterinistically fill the space to a predetermined amount. This obscures the position of the node in the DAG. The bare filter \(above\) is of varying length, so we want to saturate it equally.

All name filters must be unique \(since storage is append-only\). We gain uniqueness by including the revision number in XOR filter. However, we only want to reveal revision numbers to authorized users. We use the AES key itself as a cryptographic pepper, and append to the version, and add that hash to the XOR filter.

```haskell
pepperedRevision = hash (revision <> aesKey)
versionedNameFilter = bareFilter .&. peppredRevision
```

Saturation is then achieved by iteratively hashing the filter into its successor until a certain number of bits are set to 1.

```haskell
makeNameFilter :: XORFilter
makeNameFilter = saturate aesKey 320 (bareFilter .&. pepperedRevision)

saturateTo :: Natural -> AES256 -> XORFilter -> XORFilter
saturateTo threshold key xorFilter =
  if Binary.sum xorFilter >= threashold
    then xorFilter
    else saturate aesKey threshold (xorFilter .&. step)
    
  where
    step = hash (key <> xorFilter)
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

## Append Access

WNFS is \(normally\) nondestructive. The one exception is that the root user has the ability to overwrite the entire fil system by updating the root anchor for their entire WNFS \(e.g. the DNSLink for `${username}.fission.name`\). Under normal operation, WNFS ony allows appending to the public, private, and pretty sections. Being append-only, all private nodes have unique names that prevent overwriting.

Append \(or ”write”\) access is handled via UCANs. WNFS uses path-based permissions, so if a user has write permissions to `/photos/vacation`, they also have write permissions to `/photos/vacaction/beach`. In the public section, this is done by checking that the path being written is prefixed with a path.

Paths are completely obscured in the private section. UCANs reference some bare name filter \(described above\) that must be be a match \(binary OR\) for the filter being suggested. This does leak how much of the graph a user is allowed to write to, since the higher in the DAG they have access to, the fewer bits are set in the bare filter. However, and astute 

This check is extremely quick for anyone the verifier, who may need to check a large number of updates submitted in addition to Merkle proofs that nothing else has altered besides these additions.

### Moved Nodes

Since nodes are not aware of the paths to them \(aside from their bare name filter\), a user with append access may move a node to another part of the DAG that they have append access to \(much like in the public section\).

Instead of a simple removal of a link and additon to the latest generation elsewhere, the writer has the option to signpost this change with a redirection node \(again, like the public section\).

The wrinkle is that the moved node now MUST be encrypted with a new key, and given a new bare name filter. The signposting if for users that have access to both portions of the DAG, either through a shared parent, or with multiple read keys. The new name filter is obvious both for simple consistency, and to ensure that only users authorized for this part of the DAG can write to its children \(which may have a very different semantic meaning in the new position\).

