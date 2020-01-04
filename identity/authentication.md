---
description: Proof that the user performing the action is who they say they are
---

# Authentication

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



