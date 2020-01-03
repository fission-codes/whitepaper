# Overview

## Abstract

User authentication is provided by signing messages with an asymmetric key pair. For maximum comparability, this is then wrapped in a [W3C-specified DID document](https://www.w3.org/TR/did-core/).

Fission uses the _Networking and Cryptography library_ [\(`nacl`\) DID method](https://github.com/uport-project/nacl-did).

## Content Address

Since a Fission DID does not change, it may be referenced at a stable CID.

## Elliptic Curve

Following [Postel's Law](https://lawsofux.com/postels-law), Fission accepts many key signing schemes, but only generates keys on [Curve 25519](https://cr.yp.to/ecdh.html), with signatures on the [Edwards Curve](http://cr.yp.to/newelliptic/newelliptic.html).

We have chosen Edwards 25519 for a multitude of reasons, not least of which being reasonable performance and quantum-resistant security.

> \[...\] concretely Curve25519 works with keys consisting of about 256 bits, while an equivalent RSA instantiation would need key sizes of 3072 bits long.  
> [Source](https://www.esat.kuleuven.be/cosic/elliptic-curves-are-quantum-dead-long-live-elliptic-curves/)

Elliptic curve cryptography is by no means "perfect security", and can be defeated if the verifier does not verify that the public key actually falls on the correct curve. As such, please verify that the signature that comes in a payload is indeed on Curve 25519.

