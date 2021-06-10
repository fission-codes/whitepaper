# Append Access

WNFS is nondestructive \(under normal operation\). The one exception is that the root user has the ability to overwrite the entire file system by updating the root anchor for their entire WNFS \(e.g. the DNSLink for `${username}.fission.name`\). WNFS only allows appending to the public, private, and pretty sections. Being nondestructive, all secret nodes have unique names that prevent overwriting.

Append \(or ”write”\) access is handled via UCANs. WNFS uses path-based permissions, so if a user has write permissions to `/photos/vacation`, they also have write permissions to `/photos/vacaction/beach`. In the public section, this is done by checking that the path being written is prefixed with a path.

Paths are completely obscured in the private section. UCANs reference some bare name filter \(described above\) that must be be a match \(binary `OR`\) for the filter being suggested. This does leak how much of the graph a user is allowed to write to, since the higher in the DAG they have access to, the fewer bits are set in the bare filter. However, and astute 

This check is extremely quick for anyone the verifier, who may need to check a large number of updates submitted in addition to Merkle proofs that nothing else has altered besides these additions.

### Moved Nodes

Since nodes are not aware of the paths to them \(aside from their bare name filter\), a user with append access may move a node to another part of the DAG that they have append access to \(much like in the public section\).

Instead of a simple removal of a link and addition to the latest generation elsewhere, the writer has the option to signpost this change with a redirection node \(again, like the public section\).

The wrinkle is that the moved node now MUST be encrypted with a new key, and given a new bare name filter. The signposting if for users that have access to both portions of the DAG, either through a shared parent, or with multiple read keys. The new name filter is obvious both for simple consistency, and to ensure that only users authorized for this part of the DAG can write to its children \(which may have a very different semantic meaning in the new position\).

