# JWT Structure



{% hint style="warning" %}
What follows is the UCAN 0.4.x technical specification. This currently gets updated every few weeks, but we try to keep the API as backwards-compatible as possible.
{% endhint %}

{% hint style="info" %}
### **TL;DR**

UCAN is the familiar, standard JWT format, plus a few required keys
{% endhint %}

The JWT structure is a convenient container to carry the authenticated information we need for the functioning of a UCAN. From a purely technical perspective, there’s nothing special about the JWT. However, in terms of human factors, the JWT is currently the most common and widely understood format for tokens in web applications today.

## Uniqueness

The hash of a token may be taken as a nonce. As such, each token MUST be unique. The Fission server also requires that all tokens be used at most once to avoid replay attacks. If there is reason to make the same request multiple times in a narrow time window, it is recommended to vary the `exp` field \("expiry-overloading"\), or add an explicit `nonce`.

## Skeleton

The [JWT standard](https://tools.ietf.org/html/rfc7519) includes a number of sections and keys that are required. For those not familiar, a JWT is made up of three components:

| Section | Purpose |
| ---: | :--- |
| Header | Token Metadata |
| Payload | Authorization Information |
| Signature | Token Authentication |

The vast majority of the UCAN specification is in the body. We will note changes elsewhere as required.

## Header

The header MUST include the following fields:

| Field | Meaning | Valid Options | Required |
| :---: | :---: | :---: | :---: |
| `“alg“` | Signature Algorithm | `”EdDSA”` or `”RS256”` | ✔️ |
| `“typ“` | Type | `”JWT”` | ✔️ |
| `”ucv”` | UCAN Version | Semver \(`”m.n.p”`\) | ✔️ |

{% hint style="info" %}
Note that EdDSA is not in the JWT spec [RFC 7519](https://tools.ietf.org/html/rfc7519) at time of writing, but already widely used “in the wild“, in common JWT libraries, and is listed on [jwt.io](https://jwt.io).

EdDSA applied to JOSE \(including JWT\) exists as its own spec: [RFC 8037](https://tools.ietf.org/html/rfc8037).
{% endhint %}

### Example

```javascript
{
  "alg": "Ed25519",
  "typ": "JWT"
  "ucv": "0.4.0"
}
```

## Payload

This section describes the authorization claims being made, who is involved, and when it’s valid.

```typescript
{ 
  "iss": DID,
  "aud": DID,

  "exp": UTCTime,
  "nbf": UTCTime,
  
  "prf": [Proof],
  "att": [Attenuation],
  "fct": [Fact]
}
```

### Sender/Receiver

| Field | Long Name | Role | Required |
| :---: | :---: | :---: | :---: |
| `“iss“` | Issuer | Sender DID / signer | ✔️ |
| `“aud“` | Audience | Receiver DID | ✔️ |

{% hint style="success" %}
Being self-signed, these are the DIDs of the sender and receiver. The token signature MUST be signed with the private key associated with the `“iss”` field
{% endhint %}

#### Example

```javascript
"aud": "did:key:zStEZpzSMtTt9k2vszgvCwF4fLQQSyA15W5AQ4z3AR6Bx4eFJ5crJFbuGxKmbma4",
"iss": "did:key:z5C4fuP2DDJChhMBCwAkpYUMuJZdNWWH5NeYjUyY8btYfzDh3aHwT5picHr9Ttjq",
```

### Time Bounds

`“nbf“` and `“exp”` stand for ”not before” and ”expires at” respectively. These are standard JWT fields. Taken together they represent the time bounds for a token.

| Field | Long Name | Required |
| :---: | :---: | :---: |
| `“nbf“` | Not Before | ❌ |
| `”exp“` | Expires At | ✔️ |

The `“nbf“` field is optional \(though recommended\). When omitted, it is assumed to be currently valid. Setting this field in the future allows the sender to delay ue of a UCAN. For example, you may want someone to only be able to post something over the weekend at a hackaton, but not before.

The `“exp“` field is extremely important for a number of reasons. It is strongly encouraged to keep the time as short as  possible for a use case. For instance, when sending commands to the server, keeping it to 30 seconds is very reasonable when sending over TLS.

By limiting the time range, you lower the risk of a malicious user abusing a UCAN. There’s a balance, since if a user trusts an audience \(e.g. their personal phone\), they may not want to  reauthorize it very often.

{% hint style="danger" %}
Due to clock drift, do not expect the time bounds to be exact. At minimum assume +/- 60 seconds. While we would like to depend on a logic clock, this is not always possible, so a wall clock time keeping is required to some degree.
{% endhint %}

#### Example

```javascript
"nbf": 1529496683,
"exp": 1575606941,
```

## Facts

`“fct”` is an optional UCAN field for arbitrary facts and proofs of knowledge. These can be things like providing a raw valuet that is hased elsewhere in the UCAN, signing a challenge string with the private key associated with the `“iss”`, a Merkle proof, and so on.

To qualify as valid “facts”, they MUST be self evident & externally verifiable.

The values in this field MUST be the value directly, or individual CIDs of the facts, or a tree of CIDs. Prefer direct values whenever possible.

| Field | Long Name | Required |
| :--- | :--- | :--- |
| `“fct”` | Facts | ❌ |

An empty facts field may be represented as the absence of the field or an empty array.

#### Example

```javascript
"fct": [
  {
    "sha256": "B94D27B9934D3E08A52E52D7DA7DABFAC484EFE37A5380EE9088F7ACE2EFCDE9",
    "msg": "hello world"
  }
]
```

## Proofs

The `“prf”` section is reserved for UCAN proofs — the ”inputs” of the UCAN. Each proof MUST form a chain all the way back to the resource originator / owner. If a UCAN does not include a `“prf”` field, it is read as being the initial UCAN. In this case, the `”iss”` is the resource originator / owner for everything in the `“cap”` section.

In the case of multiple proofs, any capabilities not covered by a proof are considered to be claimed by the issuer DID.

| Field | Long Name | Required |
| :--- | :--- | :--- |
| `”prf”` | UCAN Proofs | ❌ |

This field contains an array of proofs. The field is required if you are accessing deleagted resources. If the resources are associated with the current DID, this field may be omitted.

Inline proofs MUST include the entire _encoded_ token, since they will be validated by the receiver.

{% hint style="warning" %}
These UCAN chains — especially with 2048-bit RSA DIDs — have the potential to become quite large relative to the header size limit of some servers, running roughly 2kb/layer. You can substitute them for CIDs of the proofs _as long as the proof is reachable over IPFS_. Prefer inlining the UCANs whenever possible, as choice of format will also permanently affect delegates.
{% endhint %}

#### Example

```javascript
"prf": [
  "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsInVhdiI6IjAuMS4wIn0.eyJhdWQiOiJkaWQ6a2V5OnpTdEVacHpTTXRUdDlrMnZzemd2Q3dGNGZMUVFTeUExNVc1QVE0ejNBUjZCeDRlRko1Y3JKRmJ1R3hLbWJtYTQiLCJpc3MiOiJkaWQ6a2V5Ono1QzRmdVAyRERKQ2hoTUJDd0FrcFlVTXVKWmROV1dINU5lWWpVeVk4YnRZZnpEaDNhSHdUNXBpY0hyOVR0anEiLCJuYmYiOjE1ODg3MTM2MjIsImV4cCI6MTU4OTAwMDAwMCwic2NwIjoiLyIsInB0YyI6IkFQUEVORCIsInByZiI6bnVsbH0.Ay8C5ajYWHxtD8y0msla5IJ8VFffTHgVq448Hlr818JtNaTUzNIwFiuutEMECGTy69hV9Xu9bxGxTe0TpC7AzV34p0wSFax075mC3w9JYB8yqck_MEBg_dZ1xlJCfDve60AHseKPtbr2emp6hZVfTpQGZzusstimAxyYPrQUWv9wqTFmin0Ls-loAWamleUZoE1Tarlp_0h9SeV614RfRTC0e3x_VP9Ra_84JhJHZ7kiLf44TnyPl_9AbzuMdDwCvu-zXjd_jMlDyYcuwamJ15XqrgykLOm0WTREgr_sNLVciXBXd6EQ-Zh2L7hd38noJm1P_MIr9_EDRWAhoRLXPQ"
]
```

## Attenuation

The attenuated resources \(i.e. output\) of a UCAN is an array of heterogeneous resources and capabilities \(defined below\).

The union of this array must be a strict subset \(attenuation\) of the proofs plus resources created/owned/originated by the `”iss”` DID. This scoping also includes time ranges, making the proof that starts latest and proof the end soonest the lower and upper time bounds.

Each capability has its own semantics. They consist of at least a resource and a capability, generally adhering to the form:

```javascript
{
  $TYPE: $IDENTIFIER,
  "cap": $CAPABILITY
}
```

| Field Name | Long Name | Required |
| :--- | :--- | :--- |
| `“att“` | Attenuation | ✔️  |

#### Example

```javascript
"att": [
  {
    "wnfs": "boris.fission.name/public/photos/",
    "cap": "OVERWRITE"
  },
  {
    "wnfs": "boris.fission.name/private/4tZA6S61BSXygmJGGW885odfQwpnR2UgmCaS5CfCuWtEKQdtkRnvKVdZ4q6wBXYTjhewomJWPL2ui3hJqaSodFnKyWiPZWLwzp1h7wLtaVBQqSW4ZFgyYaJScVkBs32BThn6BZBJTmayeoA9hm8XrhTX4CGX5CVCwqvEUvHTSzAwdaR",
    "cap": "APPEND"
  },
  {
    "email": "boris@fission.codes",
    "cap": "SEND"
  }
]
```

### Resources

#### Resource Type

This merely a unique identifier to indicate the type of thing being being described. For example, `“wnfs“` for the WebNative File System, `“email”` for email, and `“domain“` for domain names.

#### Resource Identifier

This value depends on the resource type. A resource identifier is a canonical reference of some sort. For instance, a DNSLink, email address, domain name, or username.

These values may also include the wildcard \(`*`\). This means ”any resource of this type”, even if not yet created, bounded by proofs. These are generally used for account linking. Wildcards are not required to delegate longer paths, as paths are generally taken as `OR` filters.

| Resource Value | Meaning |
| :--- | :--- |
| `"*"` | Delegate all resources of any type that are in scope |
| `{"wnfs": "/file/path/"}` | File paths in our file system |
| `{"app": "*"}` | All apps that the `iss` has access to, including future ones |
| `{"app": "myapp.fission.app"}` | A URL for the app \(ideally the autoassigned one\) |
| `{"domain": "*"}` | All domain names that a user has imported or bought |
| `{"domain": "somedomain.com"}` | A domain name that a user has imported or bought |

