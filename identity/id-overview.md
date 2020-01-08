# Overview

## Abstract

User authentication is provided by signing messages with an asymmetric key pair. For maximum comparability, this is then wrapped in a [W3C-specified DID document](https://www.w3.org/TR/did-core/).

Fission uses the [DID `key` method](https://digitalbazaar.github.io/did-method-key/). These can be though of as immutable DID documents, or simply as cryptographic key pairs.

Fisison makes a strict distinction between authentication and authorization. An identity has implicit right associated with it: it has "super user" access to things that it owns. It can delegate various forms of access to other keys.

## Content Address

A Fission DID does not change, both the public key and the DID document may be referenced at stable CIDs.

## Elliptic Curve

Following [Postel's Law](https://lawsofux.com/postels-law), Fission accepts many key signing schemes, but only generates keys on [Curve 25519](https://cr.yp.to/ecdh.html), with signatures on the [Edwards Curve](http://cr.yp.to/newelliptic/newelliptic.html).

We have chosen Edwards 25519 for a multitude of reasons, not least of which being reasonable performance and quantum-resistant security.

> \[...\] concretely Curve25519 works with keys consisting of about 256 bits, while an equivalent RSA instantiation would need key sizes of 3072 bits long.  
> [Source](https://www.esat.kuleuven.be/cosic/elliptic-curves-are-quantum-dead-long-live-elliptic-curves/)

Elliptic curve cryptography is by no means "perfect security", and can be defeated if the verifier does not verify that the public key actually falls on the correct curve. As such, please verify that the signature that comes in a payload is indeed on Curve 25519.

