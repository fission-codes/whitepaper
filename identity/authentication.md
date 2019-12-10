---
description: Proof that the user performing the action is who they say they are
---

# Authentication

## DIDs

User authentication is provided by 

Fission IDs conform to the W3C DID spec, but only requires a message signed with the associated private key to authenticate.

### Method

Fission uses the `nacl` DID method

### Unique Key

Unlike other DID methods, a Fission Identity \(FID\) MUST contain _exactly one_ key. Fission treats identity as pseudonymous at best, and thus regelates access control to the realm of verifiable claims.

### Content Address

Since a Fission DID does not change, it may be referenced at a stable CID.

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

## Curve25519 & Edwards25519

Following Postel's Law, the Fission Web API accepts any key signing scheme, but only generate Curve25519 keys. The Edwards Curve is ...

## JWT Authentication



{% hint style="info" %}
The IETF Network Working Group has a work-in-progress spec for a type of signature-based authentication. This spec has been in progress since 2013, and is not finalized. This serves our needs better than shoehorning in signature authentication on top of JWTs, and gives us a more aligned approach to authentication. We may move to this method in the future.
{% endhint %}

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
  "nonce": 
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
  data: 'eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NkstUiJ9.eyJpYXQiOjE1NzE2OTIyMzMsImV4cCI6MTk1NzQ2MzQyMSwiYXVkIjoiZGlkOmV0aHI6MHhmM2JlYWMzMGM0OThkOWUyNjg2NWYzNGZjYWE1N2RiYjkzNWIwZDc0IiwibmFtZSI6InVQb3J0IERldmVsb3BlciIsImlzcyI6ImRpZDpldGhyOjB4ZjNiZWFjMzBjNDk4ZDllMjY4NjVmMzRmY2FhNTdkYmI5MzViMGQ3NCJ9'
}
```

### Fields

#### Nonce

The signature MUST include a monotonically-increasing nonce.

The use of a nonce in conjunction with a timestamp ensures that a man-in-the-middle attack cannot do a fast-follow replay. Montonically increasing nonces 

#### Timestamp

A request SHOULD include the following timestamps:

* `created`
* `expires`

{% embed url="https://www.w3.org/TR/webauthn/" %}



