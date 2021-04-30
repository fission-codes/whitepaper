# Revocation

## Removal

Revocation is possible by either changing the address of the data \(e.g. by adding a nonce\), by changing the encryption key, or ideally both. This is laregly a moot distinction, since encrypting data with a new key changes the protocol-level data \(the encrypted blocks\), and thus the address.

If another user has a copy of the data being revoked, either by having it on their local disk or with a hard link to the same CID, they will have continued access to this data. This is the same as having downloaded a copy of the data previously — we can’t reach into someone else’s computer and delete files.

## Redistribution

Note that removal of access via rotation is a very wide action. Sub-deleation of read access of that data when not in a child relationship needs to be re-granted.

Once a new key is used for the data, authorized users need to be granted the new key.

### WNFS

The Web Native File System handles this via the `shared_by_me` and `shared_with_me` segments. If there’s a ”cache miss” in your `shared_with_me`, you may check the target file system’s `shared_by_me` segment, and look up the new key.

This extends to other resources as well. The WNFS happens to be an effective data-layer substrate for key exchange broadly.

