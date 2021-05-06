# Query Access

If the user has access to a reference to some data — and the correct decryption key — they have the ability to read the data. This is correct-by-construction: there mere fact of having this tuple confers read access. In the case of unencrypted data, you may think of the decryption key as being the identity function.

### Elements

1. Reference \(e.g. content address\)
2. Decryption key \(e.g. AES256-GCM symmetric key\)

## WNFS & Cryptrees

The Web Native File System \(AKA ”WNFS” — found in its own section of this whitepaper\) is built on top of this style of read access, by using a technique known as “[cryptrees](https://raw.githubusercontent.com/ianopolous/Peergos/master/papers/wuala-cryptree.pdf)“.

In the private section, each directory contains pointers and keys for all of its children. As such, granting access to a directory also grants access to all of its children. This is granted “merely” by the parent relation.

## Sharing

Granting another user, machine, app, or browser tab read access to some resource \(e.g. WNFS\) is as simple as handing them a copy of the key and a pointer to the relevant data.

This is self-contained information, and transport agnostic — any secure channel works. In Fission’s system, this is generally handled directly in WNFS via the `shared_by_me` and `shared_with_me` segments. We also transfer data via a bootstrapped secure channel on top of a WebRTC-based pubsub, and even directly in query parameters.

{% hint style="danger" %}
Note that anyone with read access can also grant the same access to others without alerting the owner. Revocation exists in the current spec \(next section\), but methods for more granular access control exist, though they have tradeoffs in availability, time, storage, or all three. We believe that in practice this is not required for the 98% use case, and that key rotation is sufficient. Different protocols can be implemented on top of this layer if needed.
{% endhint %}

## Revocation

### Removal

Revocation is possible by either changing the address of the data \(e.g. by adding a nonce\), by changing the encryption key, or ideally both. This is largely a moot distinction, since encrypting data with a new key changes the protocol-level data \(the encrypted blocks\), and thus the address.

If another user has a copy of the data being revoked, either by having it on their local disk or with a hard link to the same CID, they will have continued access to this data. This is the same as having downloaded a copy of the data previously — we can’t reach into someone else’s computer and delete files.

### Redistribution

Note that removal of access via rotation is a very wide action. Sub-deleation of read access of that data when not in a child relationship needs to be re-granted.

Once a new key is used for the data, authorized users need to be granted the new key.

### WNFS

The Web Native File System handles this via the `shared_by_me` and `shared_by_others` segments. If there’s a ”cache miss” in your `shared_by_others`, you may check the target file system’s `shared_by_me` segment, and look up the new key.

This extends to other resources as well. The WNFS happens to be an effective data-layer substrate for key exchange broadly.

