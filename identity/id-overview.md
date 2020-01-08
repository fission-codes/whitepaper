# Overview

## Abstract

User authentication is provided by signing messages with an asymmetric key pair. For maximum compatability, this is then wrapped in a [W3C-specified DID document](https://www.w3.org/TR/did-core/).

Fission uses the [DID `key` method](https://digitalbazaar.github.io/did-method-key/). These can be though of as immutable DID documents, or simply as cryptographic key pairs.

Fission makes a strict distinction between authentication and authorization. An identity has implicit rights associated with it: it has "super user" access to things that it owns. It can delegate various forms of access to other keys, both by re-sharing keys, [macaroons](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/41892.pdf) for delegated access controlled services, and other forms of self-signing certificates.

Our platform does not currently need to know that two keys are the same underlying user, and is focused on correct-by-construction access regulated by having access to keys.

Given that these keys are so simple and immutable, they can be composed by more complex DID methods at a later time.

## Content Address

A Fission DID does not change, both the public key and the DID document may be referenced at stable CIDs.

