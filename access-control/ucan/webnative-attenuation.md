# WebNative Attenuation

Attenuations for the WebNative File System are a thin layer on top of the UCAN spec presented earlier in this paper.

The resource type is `”wnfs”`. The resource value is a DNSLink pointing at the highest node in the graph granting access. Everything below is given the same acces \(as its content\).

## Paths

### Public



## Capabilities

WNFS capabilities are monotone, where each level “contains” the capabilities below it.

### 1. CREATE

Append access allows for the adding of completely new file paths, but not new generations.

### 2. REVISE

Create new revisions of existing files. Viewed from another angle, this is an append-only permission on a persistent structure.

Includes Capability 1.

### 3. SOFT\_DELETE

Remove a file path from the next generation. The file will still be available in the WNFS history.

Includes Capability 1 and Capability 2.

### 4. OVERWRITE

> With great power comes great responsability  
>   
> — Uncle Ben via Spider Man

The ability to rewrite history. This is a fairly dagerous opration, as it may break assumptions from other users. This is the “hard delete”, contrasted with Capability 3’s “soft delete”.

Includes Capability 1, Capability 2, and Capability 3.

### 5. SUPER\_USER

> Property rights are theoretical socially-enforced constructs in economics for determining how a resource or economic good is used and owned. \[...\] the right to transfer the good to others, alter it, abandon it, or destroy it \(the right to ownership cessation\)  
>   
> — Wikipedia, [Property Rights \(economics\)](https://en.wikipedia.org/wiki/Property_rights_%28economics%29)

The ability to destroy the file system itself, or transfer it to another owner. “Transfer here is the transfer of ownship over the actual head pointer, as all other data can be copied with sufficient read permissions.

Includes Capability 1, Capability 2, Capability 3, and Capability 4.

