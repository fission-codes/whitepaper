# File System

## Resource

The resource type is `”wnfs”`. The resource value is a DNSLink pointing at the highest node in the graph granting access. Everything below is given the same acces \(as its content\).

## Paths

All paths in WNFS are given in relationship to some head pointer via a [DNSLink](https://docs.ipfs.io/concepts/dnslink/). Access is given by a binary `OR` of the path, and match that to the longer path. If they match, access is granted.

Trailing slashes on directories are OPTIONAL, though recommended for clarity. The verifier MUST infer a trailing slash. This is to prevent matching `foo/barbaz` with `foo/bar`.

### Public

Public paths are human readable, and are fairly straightforward. The binary `OR` mentioned above works out to a prefix. As such `boris.fission.name/foo/` matches `boris.fission.name/foo/bar.jpg`, but not `boris.fission.name/nope/bar.jpg`  or `icidasset.fission.name/foo/bar.jpg`.

### Private

Private paths have an additional wrinkle. For privacy, paths are given as namefilters — Bloom filters of the AES keys of the path, plus some filler. The resource path is given as the namefilter without the additional filler — a ”bare” namefilter.

We follow the same procedure — check if the path successfully `OR`s with the bare namefilter.

For space saving reasons, these namefilters are then hashed down for storage. The UCAN is required to also include a mapping “fact“ of the namefilter and its base58 SHA256 value.

## Capabilities

WNFS capabilities are monotone, where each level “contains” the capabilities below it.

### 1. CREATE

At the platform layer, this ”create new path”. This is a new path _relative_ to the most recent generation. So, if this path existed in a previous generaion, but was then removed, it’s allowed to be created anew.

Append access allows for the adding of completely new file paths, but not new generations. In other words, at the low-level protocol layer, this is a restricted form of “append only”.

### 2. REVISE

Create new revisions of existing files. Viewed from another angle, this is an append-only permission on a persistent structure.

Includes Capability 1.

### 3. SOFT\_DELETE

Remove a _public_ file path from the next generation. The file will still be available in the WNFS history.

It is not possible to inspect a private file, so this capability does not make sense in the context of the private file system.

Includes Capability 1 and Capability 2.

### 4. OVERWRITE

> With great power comes great responsibility  
>   
> — Uncle Ben via Spider Man

The ability to rewrite history. This is a fairly dangerous operation, as it may break assumptions from other users. This is the “hard delete”, contrasted with Capability 3’s “soft delete”.

Includes Capability 1, Capability 2, and Capability 3.

### 5. SUPER\_USER

> Property rights are theoretical socially-enforced constructs in economics for determining how a resource or economic good is used and owned. \[...\] the right to transfer the good to others, alter it, abandon it, or destroy it \(the right to ownership cessation\)  
>   
> — Wikipedia, [Property Rights \(economics\)](https://en.wikipedia.org/wiki/Property_rights_%28economics%29)

The ability to destroy the file system itself, or transfer it to another owner. “Transfer" here is the transfer of ownership over the actual head pointer, as all other data can be copied with sufficient read permissions.

Includes Capability 1, Capability 2, Capability 3, and Capability 4.

