# Revoking

## Removal

Revocation is possible by either changing the address of the data \(e.g. by adding a nonce\), by changing the encryption key, or ideally both. This is laregly a moot distinction, since encrypting data with a new key changes the protocol-level data \(the encrypted blocks\), and thus the address.

If another user has a copy of the data being revoked, either by having it on their local disk or with a hard link to the same CID, they will have continued access to this data. This is the same as having downloaded a copy of the data previously — we can’t reach into someone else’s computer and delete files.

## Redistribution

Note that removal of access via rotation is a very wide action. Subdeleation of read access of that data when not in a child relationship needs to be re-granted.

Once a new key is used for the data, authorized users need to be alerted.

