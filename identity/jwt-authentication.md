# JWT Authentication

To guard against replay attacks, all authenticated requests to and from Fission are one-time use only, and feature a unique nonce inside a sliding time window.

Fission uses the familiar JWT format, plus a few additional keys.

## Complete Example

```javascript
{
  "alg": "Ed25519",
  "typ": "JWT"
}
{ 
  "iss": "did:key:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI#pubkey",
  "sub": "_did.snak.fission.name",
  "aud": "_did.runfission.com",
  "nbf": 1529496683,
  "exp": 1575606941,
  "method": "GET"
  "path": "/users/snak"
  "query": "fname=satoshi&lname=nakamoto",
  "bodyDigest": "0a7c05d1b06a76f7e76fcc74d8606518e6a916368b92f1f622db3a8c824c9f1f"
}
SIGNATURE
```

## Header

The header MUST contain the following fields:

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

The following claims MUST be included: 

* `"iss"`
* `"sub"`
* `"aud"`
* `"nbf"`,
* `"exp"`

The JWT can OPTIONALLY include any or all of the following:

* `"method"`
* `"path"`
* `"params"`
* `"paramDigest"`
* `"bodyDigest"`

Additional claims MAY be included, as required by additional applications or ptorocols.

### `iss`

Since Fission JWTs are self-signed, so the the "issuer claim" \(`iss`\) MUST specify the DID public key used to sign the token. This key MUST be formatted in the `key` DID scheme. The key MUST be prefixed with `did:key:`, and end in `#pubkey`.

#### Example

```javascript
"iss": "did:key:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI#pubkey"
```

### `sub`

The JWT subject claim \(`sub`\) MUST specify the the subject that the JWT is acting on behalf of. This MUST be either:

1. Recommended: the subject's DID
2. Fully-qualified [TXT record DID name](https://tools.ietf.org/html/draft-mayrhofer-did-dns)

This is typically the same as the `iss`, but MAY be different — for instance in cases of delegated authorization. The direct DID is preferred as it does not rely on any external systems.

#### Key Example

```javascript
  "sub": "did:key:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI#pubkey",
```

#### DNS Example

```javascript
  "sub": "_did.expede.fission.name",
```

### `aud`

The JWT audience \(`aud`\) MUST be one of:

1. Recommended: the DID of the intended recipient
2. [TXT-record based DID name](https://tools.ietf.org/html/draft-mayrhofer-did-dns)
3. In the case of an HTTP service: the domain name unprefixed by HTTP scheme

The DID is preferred as it does not rely on any external systems.

#### DID Example

```javascript
"aud": "did:key:u49kIt8edfomL3FtQ2ngnGZtNE4UmVCABuhnLyAsYDi#pubkey"
```

#### DNS Example

```javascript
"aud": "_did.fission.codes"
```

#### HTTP Service Example

```javascript
"aud": "runfission.com"
```

### `nbf`

The Unix time at which the token becomes valid, expressed in milliseconds. This is typically the time it was issued, but may also be in the future.

#### Example

```javascript
"nbf": 1529496683
```

### `exp`

The Unix time at which the token is no longer valid. 

#### Example

```javascript
"exp": 1575606941
```

## Additional Notes

The hash of a token may be taken as a nonce. As such, every request MUCT be unique. If there is reason to make the same request multiple times in a narrow time window, it is recommended to vary the `exp` field \("expiry-overloading"\), or add an explicit `nonce`.

### Signature Authentication

Authentication method will be extensible in the future, ideally converging to the IETF's Signature method as that becomes a more mature standard.

{% embed url="https://www.w3.org/TR/webauthn/" %}



