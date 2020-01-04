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
  "iss": "did:nacl:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI",
  "aud": "https://runfission.com",
  "sub": "/ipfs/Qm12345"
  "iat": 1529496683,
  "exp": 1575606941,
  "nonce": 12345,
  "digest": "a0Bc2De3Fg4Hi5Jh6"
}
SIGNATURE
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



