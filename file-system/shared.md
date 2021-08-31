# Share

There are three cases where we need to exchange data in an offline manner. Fundamentally these are all variants on ”I need to make this update, but don’t have the other party online.”

## Exchange & Share Keys

Sharing information with a user that’s offline is easy thanks to authenticated key exchange. All WNFS users widely distribute a list of public 2048-bit RSA public keys — their non-exportable ”exchange keys” — as DIDs at a well-known location \(`/public/.well-known/exchange/*`\). These RSA keys are used to send a symmetric key to a recipient; the "share key". This symmetric key is then used to decrypt pointers, UCANs, and other messages.

As there's only a single recipient per key, there is no need for backwards secrecy, and so we can use a simple natural number to represent the version, and to do seek-head.

## Shared By Me

### Layout

This is a one-to-many exchange. Because of how account linking works, any given user will typically have a small number of exchange keys \(in the range of 1 to 5\). Each user only has access to a single key at any given time, so the sender will use a single key to share with multiple recipient keys:

![](../.gitbook/assets/screen-shot-2021-06-10-at-13.02.58%20%281%29.png)

The actual file pointers \(in grey above\) are only generated once per permissions group. Encrypting the share key with a group is done per key \(if shared with 5 people, then 5 nodes containing a share ket will be created\). This "share key" gives read access to a SNode at a special namefilter address \(see below\). There is nothing unusual about the content of this SNode: it is a directory contains named pointers and keys for other nodes in the private section.

Since all data is immutable-by-default, updating the share key is done by creating a new key and placing it at the incremented version number.

### Top-Level Lookup Namefilter

The recipient needs a way to deterministically look up their node in the namefilter, without giving away the list of everyone that has been shared with. To facilitate direct copying for "shared with me", this should also be globally unique.

To accomplish this, we salt the recipient's root DID with the sender's root DID, and the version number, and then take the hash.

```typescript
// Top-level lookup by recipient's root DID
const shareNameFilter =
  (recipientRootId: Did, senderRootId: Did, version: number): Namefilter => {
    const filter = new Namefilter()
    return filter
      .append(sha256(`${receiver}${sender}${version}`))
      .saturate()
  }
```

### Payload

The content of these files is merely a pointer and the requesite key. The only requirement for the associated namefilter is that it be in the private partition.

{% hint style="info" %}
This may become serialized as an official [multiformat](https://multiformats.io/) in the future
{% endhint %}

```typescript
interface SharedKeyPayload {
  algo:       KeyType;    // e.g. "AES-256"
  key:        Uint8Array; // Raw key bytes
  namefilter: Namefilter; // Saturated namefilter as bytes
}
```

### Entry Point

Any node in the private partition may be pointed to. In the simple case, this is an existing file. However, in the case that the user is being given a UCAN, or multiple points to many parts of the filesystem, then a entry index must be created.

This node is not versioned, since an arbitrary sharer may not have access to the key of a previous version, and thus a ratchet doesn't work. As such, we conflate the node's encryption key as its inumber, and an arbitrary bare filter \(i.e. one that the writer is allowed to write with\) as the parent filter:

```typescript
// Namefilter for the entry index
const entryIndexNamefilter = 
  (shareKey: AesKey, bareFilter: Namefilter): Namefilter =>
    bareFilter.append(sha256(shareKey)).saturate()
```

## Shared With Me

The inverse of "shared by me" is "shared with me". Here the root user exchange keys are the recipient. This is used for two reasons:

1. Copying keys to your own file system to ensure that you have a copy in the case that the original file system gets overwritten
2. Distribution to other linked devices

This looks nearly identical to the "shared by me" section.

