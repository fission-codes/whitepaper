---
description: 'Your data, everywhere'
---

# Unified Storage

Data is at the heart of all computing, from the smallest function to the largest application. 

Content addressing focuses on data rather than physical location.

## Files

Static files are very straightforwardly added to IPFS and may be additionally labeled with metadata, linked, and so on. The only constraint is that it is impractically difficult to create a loop; we are limited to a directed acyclic graph \(DAG\).

These include all sorts of assets and links, not limited to images, static websites, packages, &c

## Database

IPFS can be used as a substrate for CRDTs. These in turn can be used for passing around information for distributed databases.

## Private Data on Public Networks

On the public IPFS network, any addressable file is available to anyone. If a file on a private network is leaked, it is addressable with the same address. We assume that the transport pipes are broken, and provides key distribution and rotation mechanisms to mediate access.

## Temporal Data Stores

No one likes it when their data disappears simply because they pressed the wrong button. Storage is cheaper all the time, and there are techniques for storing only the changed parts of some data. Being able to rewind time, recover old files, and compare changes is not only useful, but essential in a distributed setting.

There is one user-centric scenario when you would want to change the history: if some sensitive data was published publicly by accident.

