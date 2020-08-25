# Revoking

## Removal

Revocation is possible by either changing the address of the data \(e.g. by adding a nonce\), by changing the encryption key, or ideally both. This is laregly a moot distinction, since encrypting data with a new key changes the protocol-level data \(the encrypted blocks\), and thus the address.

If another user has a copy of the data being revoked, either by having it on their local disk or with a hard link to the same CID, they will have continued access to this data. This is the same as having downloaded a copy of the data previously — we can’t reach into someone else’s computer and delete files.

## Redistribution

Once keys are 

