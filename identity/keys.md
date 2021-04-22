# Keys

## Elliptic Curve

Following [Postel's Law](https://lawsofux.com/postels-law), Fission accepts many key signing schemes but only generates and 2048-bit [RSA](https://en.wikipedia.org/wiki/RSA_%28cryptosystem%29) key pairs and [Curve 25519](https://cr.yp.to/ecdh.html) keys with signatures on the [Edwards Curve](http://cr.yp.to/newelliptic/newelliptic.html) \(also known as Ed25519\).

We have chosen Ed25519 for many reasons, not least of which being reasonably good performance and quantum-resistant security.

> \[...\] concretely Curve25519 works with keys consisting of about 256 bits, while an equivalent RSA instantiation would need key sizes of 3072 bits long.  
> [Source](https://www.esat.kuleuven.be/cosic/elliptic-curves-are-quantum-dead-long-live-elliptic-curves/)

Elliptic curve cryptography is by no means "perfect security". It can be defeated if the verifier does not verify that the public key falls on the correct curve. As such, please verify that the signature that comes in a payload is indeed on Curve 25519.

