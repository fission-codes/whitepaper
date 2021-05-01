# Self-Certified

At time of writing, only certain keys are permitted: 2048-bit RSA keys and Ed25519.

## 2048-bit RSA

Despite the large key size, RSA is the only widely-trusted asymmetric key algorithm available in the [WebCrypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API).

## Curve-25519 / Ed25519

[Curve 25519](https://cr.yp.to/ecdh.html) keys with signatures on the [Twisted Edwards curve](http://cr.yp.to/newelliptic/newelliptic.html), also known as Ed25519 or EdDSA. Ed25519 was chosen for many reasons, not least of which being reasonably good performance and quantum-resistant security.

> \[...\] concretely Curve25519 works with keys consisting of about 256 bits, while an equivalent RSA instantiation would need key sizes of 3072 bits long.  
> [Source](https://www.esat.kuleuven.be/cosic/elliptic-curves-are-quantum-dead-long-live-elliptic-curves/)

Elliptic curve cryptography is by no means perfectly secure. It can be defeated if the verifier does not verify that the public key falls on the correct curve. As such, please verify that the signature that comes in a payload is indeed on the specified curve.

