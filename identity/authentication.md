# Authentication

To guard against replay attacks, all authenticated requests to and from Fission are one-time use only.

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
  "iss": "did:key:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI#pubkey",
  "sub": "expede.fission.name",
  "aud": "runfission.com",
  "sub": "/ipfs/Qm12345"
  "nbf": 1529496683,
  "exp": 1575606941, -- overloaded
  "digest": "X48E9qOokqqrvdts8nOJRJN3OWDUoyWxBf7kbu9DBPE="
}
SIGNATURE
```

## Header

The JOSE header MUST contain the following fields:

### `typ`

The `typ` MUST be set to `"JWT"`

```javascript
"typ": "JWT"
```

### `alg`

The `alg` filed MUST specify the Ed25519 [EdDSA](https://tools.ietf.org/html/rfc8032) signature scheme:

```javascript
"alg": "Ed25519"
```

## Claims

The following claims MUST be included: `"iss"`, `"sub"`, `"aud"`, `"sub"`, `"nbf"`, `"exp"`, and `"digest"`. The JWT MAY contain additional claims.

### `iss`

Since Fission JWTs are self-signed, so the the "issuer claim" \(`iss`\) MUST specify the DID public key used to sign the token. This key MUST be formatted in the `key` DID scheme. The key MUST be prefixed with `did:key`, and end in `#pubkey`.

```javascript
"iss": "did:key:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI#pubkey"
```

One acceptable approach is to use the current timestamp represented in nanoseconds. This provides an easily accessible reference that doesn't need to be persisted between devices or sessions.

Nanosecond resolution is more granular than available on some systems, but this keeps the magnitude consistent. The additional space in these cases may be filled arbitrarily; all the matters is that each value is strictly greater than the last. For instance, in JavaScript we may receive two equal time stamps, so an acceptable workaround is to use a session counter added to the timestamp to force ordering.

```javascript
let sessionCounter = 1

const now          = new Date() // e.g. 1578628940257
const milliseconds = now.valueOf()
const nanoseconds  = milliseconds * 1000000
const final        = nanoseconds + sessionCounter * 1000

sessionCounter++

return final  // e.g. 1578628940257001000
```

Fission has a slides grace window for nonces, 

#### Timestamp

A request SHOULD include JWT timestamps:

* `iat`
* `exp`

{% embed url="https://www.w3.org/TR/webauthn/" %}



