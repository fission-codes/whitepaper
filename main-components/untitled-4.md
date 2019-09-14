# Storage

Storage is crutial to any application.

Content addressing focuses on data rather than physical location.

## Files

Static files are very straigtforwardly added to IPFS, and may be additionally labelled with metadata, linked, and so on. The only constraint is that it is impractically difficult to create a loop; we are limited to a directed acyclic graph \(DAG\).

This includes all sorts of assets asnd links, not limited to images, static websites, packages, &c

## Database

IPFS can be used as a substrate for CRDTs. These in turn can be used for passing around information for distributed databases.

One such project is OrbitDB.

## Private Data on Public Networks

On the public IPFS network, any addressable file is available to anyone. Encrypting private files

