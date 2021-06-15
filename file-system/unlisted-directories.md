---
description: Public inside private
---

# Unlisted

Unlisted files are cleartext, public vnodes stored in the private file system. The primary difference is that their their root vnodes are named merely by their nonce ID and numerical version. 

You can think of these as a public forest embedded at random positions inside the McTrie. The roots behave largely like private nodes \(see below for more detail\), but their children behave exactly like public nodes.

## Names & Versioning

Much like private nodes, unlisted nodes also have a randomly generated, 256-bit nonce i-number. Since backwards secrecy is not required, versioning is achieved with a natural number instead of a cryptographic ratchet:

```haskell
barename :: INumber -> Natural -> SAH256
barename inumber version =
  Namefilter.empty
    |> Namefilter.insert inumber
    |> Namefilter.insert versionHash
    |> Namefilter.saturate
    
  where
    uuidHash    = sha256 uuid
    versionHash = sha256 (uuid <> version)
```

History link structure below the root is achieved by the same structural sharing mechanism as in the public partition. Multivalues are used during merge operations.

## Read Access

Read access is trivial: the nodes themselves are in cleartext. A user will need a pointer to the CID.

## Write Access

Write access is mediated by listing the nonce and an optional path. For example:

```javascript
{
  wnfs: {
    unlisted: "/private/mmtvrJTSHKh4cIaHg0dYps69FoxNA87EP3xTUTR3r4I=/photos/vacation/"
  }
}
```



