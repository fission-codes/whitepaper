---
description: Proof that the user performing the action is who they say they are
---

# Authentication

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

## DID Document

Since the NaCl method is a single, public key, always on the Edwards curve, the DID document itself may be reconstructed from a public key at any time. This layer exists purely for compatibility purposes.

### Keys

#### `@context`

The `@context` key simply states which version of the spec this DID conforms to.

For example: `'https://w3id.org/did/v1'`

#### `id`

The `id` is the concatenation of the method `did:nacl:` with the public key.

For example

```javascript
'did:nacl:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI'
```

#### `publicKey`

The public key array MUST:

* Contain _exactly one_ key
* Be encoded in base64 \(for transmission and consistency\)
* Follow this format:

```javascript
{
  id: 'did:nacl:myPublicKey',
  type: 'ED25519SignatureVerification',
  owner: 'did:nacl:myPublicKey',
  publicKeyBase64: 'myPublicKey'
}
```

{% hint style="warning" %}
This contains a lot of redundant information, but is required to be compatible with other DID services
{% endhint %}

Unlike other DID methods, a Fission Identity MUST contain _exactly one_ key. Fission treats identity as pseudonymous at best, and thus relegates access control to the realm of verifiable claims.

> Motivation. There is a need for non updateable DID's for use in IOT and other applications, where lack of network, size of code base and other such concerns are paramount to adoption. These concerns need to be addressed while not lowering the overall security guarantees.
>
> ~The [uPort NaCL README](https://github.com/uport-project/nacl-did)

Our motivation is focused primarily on universality. Rather than updating a document over time, the DID is minimal, immutable, and may be reconstructed from a key pair at any time.

#### `authentication`

The `authentication` array MUST:

* Contain exactly one key
* This key must be the same as in the `publicKey`
* Use the index `#key1` 
* Specify the `ED25519SigningAuthentication` scheme

```javascript
{
  type: 'ED25519SigningAuthentication',
  publicKey: `did:nacl:myPublicKey#key1`
}
```

### Example Document

```javascript
{
  '@context': 'https://w3id.org/did/v1',
  id: 'did:nacl:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI',
  publicKey: [
    {
      id: `did:nacl:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI#key1`,
      type: 'ED25519SignatureVerification',
      owner: 'did:nacl:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI',
      publicKeyBase64: 'Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI'
    }
  ],
  authentication: [
    {
      type: 'ED25519SigningAuthentication',
      publicKey: `did:nacl:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI#key1`
    }
  ]
}
```

## JWT Authentication

Fission uses the familiar JWT format, plus a few additional keys.

### Signature Authentication

Authentication method will be extensible in the future, ideally converging to the IETF's Signature method as that becomes a more mature standard. 

### Complete Example

```javascript
{
  "alg": "Ed25519",
  "typ": "JWT"
}
{ 
  "iss": "AAAAC3NzaC1lZDI1NTE5AAAAICpuH/fqCFLbAConChyVH6rZzSaxlnHSwQk6qvtPsf5E",
  "aud": "https://runfission.com",
  "sub": "/ipfs/Qm12345"
  "iat": 1529496683,
  "exp": 1575606941,
  "nonce": 12345
}
SIGNATURE





{
  header: { 
    typ: 'JWT',
    alg: 'EdDSA'
  },
  payload: {
    iat: 1571692233,
    exp: 1957463421,
    aud:  'did:nacl:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI',
    name: 'uPort Developer',
    iss:  'did:ethr:0xf3beac30c498d9e26865f34fcaa57dbb935b0d74'
  },
  signature: 'kkSmdNE9Xbiql_KCg3IptuJotm08pSEeCOICBCN_4YcgyzFc4wIfBdDQcz76eE-z7xUR3IBb6-r-lRfSJcHMiAA',
  data: 'eyJ0eXAiOiJKV1bQiLCJhbGciOiJFUzI1NkstUiJ9.eyJpYXQiOjE1NzE2OTIyMzMsImV4cCI6MTk1NzQ2MzQyMSwiYXVkIjoiZGlkOmV0aHI6MHhmM2JlYWMzMGM0OThkOWUyNjg2NWYzNGZjYWE1N2RiYjkzNWIwZDc0IiwibmFtZSI6InVQb3J0IERldmVsb3BlciIsImlzcyI6ImRpZDpldGhyOjB4ZjNiZWFjMzBjNDk4ZDllMjY4NjVmMzRmY2FhNTdkYmI5MzViMGQ3NCJ9'
}
```

### Request Fields

#### Nonce

The signature MUST include a monotonically-increasing nonce. The nonce is not required to be sequential, only increasing. A counter and timestamps are both valid options, for instance.

```javascript
{
  iss: "did:nacl:myPublicKey"
  iat: 1571692233
  exp: 1957463421
  req: {   
    method: 'PUT,
    path:   '/ipfs',
    query: '',
    nonce: 12345
    bodyDigest: 'eyJ0eXAiOiJKV1bQiLCJhbGciOiJFUzI1NkstUiJ9'
  }
} 
```

#### Timestamp

A request SHOULD include JWT timestamps:

* `iat`
* `exp`

{% embed url="https://www.w3.org/TR/webauthn/" %}



