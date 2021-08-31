# Share

There are three cases where we need to exchange data in an offline manner. Fundamentally these are all variants on ”I need to make this update, but the other party is not available.”

## Exchange & Share Keys

Sharing information with a user that’s offline is easy thanks to authenticated key exchange. All WNFS users widely distribute a list of public 2048-bit RSA public keys — their non-exportable ”exchange keys” — as DIDs at a well-known location \(`/public/.well-known/exchange/*`\). These RSA keys are used to send a symmetric key to a recipient; the "share key". This symmetric key is then used to decrypt pointers, UCANs, and other messages.

## Shared Partition

`shared` is the label for the top-level partition for both incoming a outgoing shared pointers. It is structured as a map of salted and hashed with their zero-indexed root DIDs pointing to one-or-more payloads. Each of these payloads are of fixed length, since they're encrypted with the target's public key.

```javascript
const shared_by_me: {}
const key = sha256(`${recipientRootDid}${senderRootDid}${version}`)
shared_by_me[key] = node
```

### Conflicts

If during a merge there are two shared nodes with the same value, we add one level of nesting:

```javascript
const shared_by_me: {}
const key = sha256(`${recipientRootDid}${senderRootDid}${version}`)
shared_by_me[key] = [nodeA, nodeB]
```

Or, if you prefer:

```typescript
type SharedIndex = { [Sha256]: SNode | SNode[] };
```

In the conflict case, it is possible to avoid a layer of indirection by giving multiple CIDs the same name:

```typescript
{
  Data: "",
  Links: [
    {
      Hash: CID(QmWDtUQj38YLW8v3q4A6LwPn4vYKEbuKWpgSm6bjKW6Xfe),
      Name: sha(`${recipient}${sender}42`),
      Tsize: 214
    },
    {
      Hash: CID(bafyreifepiu23okq5zuyvyhsoiazv2icw2van3s7ko6d3ixl5jx2yj2yhu),
      Name: sha(`${recipient}${sender}42`),
      Tsize: 214
    }
  ]
}
```

## Write Access

Anyone with a valid UCAN granting private partition write access may use the shared partition. While this is an append-only structure, it should be considered less stable than the public or private partitions, and may include expiration and garbage collection in the future.

## Layout

Because of how account linking works, any given user will typically have a small number of exchange keys \(in the range of 1 to 5\). Each user only has access to a single key at any given time, so the sender will use a single key to share with multiple recipient keys:

![](../.gitbook/assets/screen-shot-2021-06-10-at-13.02.58%20%281%29.png)

Encrypting the share key with a group is done per key \(if shared with 5 people, then 5 nodes containing a share key will be created\). This "share key" gives read access to a SNode at a special namefilter address \(see below\). There is nothing unusual about the content of this SNode: it is a directory contains named pointers and keys for other nodes in the private section.

The entry index \(in grey above\) is stored in the private partition. The rest is in the shared partition.

Since all data is immutable-by-default, updating the share key is done by creating a new key and placing it at the incremented version number.

## Payload

The content of these files is merely a pointer and the requisite key. Due to size limitations in RSA encryption, we store a CID instead of a namefilter. The only requirement for the associated CID points to a file that itself has namefilters in the correct private file system. This entry point node will contain pointers to one or more namefilters, ensuring that paths are relative to the current root.

{% hint style="info" %}
This may become serialized as an official [multiformat](https://multiformats.io/) in the future
{% endhint %}

```typescript
interface SharedKeyPayload {
  algo: KeyType;    // e.g. "AES-256"
  key:  Uint8Array; // Raw key bytes
  cid:  CID;        // Direct CID, not namefilter
}
```

## Entry Index

This is a regular directory SNode that contains the actual child namefilter/key pairs to more data \(regular data, UCANs, and so on\)

This node is not versioned, since an arbitrary sharer may not have access to the key of a previous version, and thus a ratchet doesn't work. As such, we conflate the node's encryption key as its inumber, and an arbitrary bare filter \(i.e. one that the writer is allowed to write with\) as the parent filter:

```typescript
// Namefilter for the entry index
const entryIndexNamefilter = 
  (shareKey: AesKey, bareFilter: Namefilter): Namefilter =>
    bareFilter.append(sha256(shareKey)).saturate()
```

## Shared With Me

The inverse of "shared by me" is "shared with me". They are stored in the same partition, named "shared". Any agent with write access to the private partition may copy data to this partition.

{% hint style="info" %}
While not strictly required, it is strongly encouraged that during copying, you also integrate keys and pointers into the private file system. This is done as normal soft links into directories where the content is relevant \(e.g. an app that works on collaborative private data\).
{% endhint %}

## Lookup & Discovery

The fundamental challenge being overcome is one of discovery: looking for data that you expect to exist. This does not solve the problem of being made aware that there is data available globally like a mailbox. However, if this information is learned out-of-band, it provides a convenient way to rendezvous.

From a UX perspective, this enables flows like what you find in document sharing applications. Users with access can grant others access and send them a link. When the recipient navigates to the document, they are immediately able to open view the document as normal.

