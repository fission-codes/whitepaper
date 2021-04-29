# Overview

## Abstract

User authentication is provided by signing messages with an asymmetric key pair. For maximum compatibility, the key pair is wrapped in a [W3C-specified DID document](https://www.w3.org/TR/did-core/).

Fission uses the [DID `key` method](https://digitalbazaar.github.io/did-method-key/). These can be thought of as immutable DID documents or simply as cryptographic key pairs. In theory, this is easily extensible to any DID type, though ones that are not self-describing generally require a network check of some kind, be it against an HTTP server or a blockchain.

Fission makes a strict distinction between authentication and authorization. An identity has implicit rights associated with it: for example, it has "super user" access to things that it owns. It can delegate various forms of access to other keys, both by re-sharing keys, [macaroons](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/41892.pdf) for delegated access-controlled services, and other forms of self-signed certificates.

The Fission platform does not currently need to know that two keys are associated with the same underlying user, and is focused on correct-by-construction access regulated by having access to keys. This means that it has a very weak concept of "identity", and is more focused on the delegation of capabilities between keys. One advantage is that aside from the originating key, there is always plausible deniability about "who" is associated with any particular key, regardless of the certificates it holds.

## Content Address

A Fission DID does not change. Both the public key and the expanded DID document may be referenced at stable CIDs.

## 

