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
| `“alg“` | Signature Algorithm | `”EdDSA”` or `”RS256”` | ✅ |
| `“typ“` | Type | `”JWT”` | ✅ |
| `”ucv”` | UCAN Version | Semver \(`”m.n.p”`\) | ✅ |

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
  "cap": [Capability],
  "fct": [Fact]
}
```

### Sender/Receiver

| Field | Long Name | Role | Required |
| :---: | :---: | :---: | :---: |
| `“iss“` | Issuer | Sender DID / signer | ✅ |
| `“aud“` | Audience | Receiver DID | ✅ |

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
| `”exp“` | Expires At | ✅ |

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

To qualify asvalid “facts”, they MUST be self evident & externally verifiable.

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

This field contains an array of proofs. The field is optional, and may be omitted.

### 

### 

### Introduction rules

## Uniqueness

When the audience is set to the Fission server, every request is required to be unique \(generally by setting a different expiry\) to prevent replay attacks.



### Resource Values

| `rsc` | Meaning |
| :--- | :--- |
| `"*"` | Delegate all resources \(includes ones in the future — for account linking\) |
| `{"floofs": "/file/path/"}` | File paths in our file system |
| `{"app": "*"}` | All apps that the `iss` has access to, including future ones |
| `{"app": "myapp.fission.app"}` | A URL for the app \(ideally the autoassigned one\) |
| `{"domain": "*"}` | All domain names that a user has imported or bought |
| `{"domain": "somedomain.com"}` | A domain name that a user has imported or bought |

You probably see the overall pattern:

* `"*"` is the wildcard "any and all"
* `{"resource_variety": "scope_of_that_resource"}`

Okay, so UCAN changes

`scp` is gone, replaced with `rsc` \("resource"\)

All other JWT claims are the same — time, potency, and so on

Inside `rsc` is an object that lists the type of resource, and the scope of that resource \(and potentially other fields in the future\)

### Some Examples

```javascript
  "rsc": { "app_url": "lucky-unicorn.fission.app" }
  "rsc": { "domain": "myawesomedomain.com" }
  "rsc": { "domain": "*" }
  "rsc": { "fission_fs": "/" }
  "rsc": "*"
```

We'll get to those `"*"` s momentarily

This layout does not put the potency, time, &c inside the `rsc`

As noted above, we do not want multiple resources in a single UCAN. The two options are "everything" or "this specific resource"

| Human | Machine |
| :--- | :--- |
| All resources | `"rsc": "*"` |
| Everything of this type of resource | `"rsc": { "domain": "*" }` |
| This specific resource | `"rsc": { "domain": "myawesomedomain.com" }` |

 Much of this will look familiar if you've done web auth in the past decade or so. Here's an example:

```text
{
  "alg": "Ed25519",
  "typ": "JWT"
  "uav": "0.1.0"
}
{
  "aud": "did:key:zStEZpzSMtTt9k2vszgvCwF4fLQQSyA15W5AQ4z3AR6Bx4eFJ5crJFbuGxKmbma4",
  "iss": "did:key:z5C4fuP2DDJChhMBCwAkpYUMuJZdNWWH5NeYjUyY8btYfzDh3aHwT5picHr9Ttjq",
  "nbf": 1588713622,
  "exp": 1589000000,
  "scp": "/"
  "ptc": "APPEND",
  "prf": null,
}
```

Example UCAN JSON Web Token

```text
Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsInVhdiI6IjAuMS4wIn0.eyJhdWQiOiJkaW
Q6a2V5OnpTdEVacHpTTXRUdDlrMnZzemd2Q3dGNGZMUVFTeUExNVc1QVE0ejNBUjZCeDRl
Rko1Y3JKRmJ1R3hLbWJtYTQiLCJpc3MiOiJkaWQ6a2V5Ono1QzRmdVAyRERKQ2hoTUJDd0
FrcFlVTXVKWmROV1dINU5lWWpVeVk4YnRZZnpEaDNhSHdUNXBpY0hyOVR0anEiLCJuYmYi
OjE1ODg3MTM2MjIsImV4cCI6MTU4OTAwMDAwMCwic2NwIjoiLyIsInB0YyI6IkFQUEVORC
IsInByZiI6bnVsbH0.Ay8C5ajYWHxtD8y0msla5IJ8VFffTHgVq448Hlr818JtNaTUzNIw
FiuutEMECGTy69hV9Xu9bxGxTe0TpC7AzV34p0wSFax075mC3w9JYB8yqck_MEBg_dZ1xl
JCfDve60AHseKPtbr2emp6hZVfTpQGZzusstimAxyYPrQUWv9wqTFmin0Ls-loAWamleUZ
oE1Tarlp_0h9SeV614RfRTC0e3x_VP9Ra_84JhJHZ7kiLf44TnyPl_9AbzuMdDwCvu-zX
jd_jMlDyYcuwamJ15XqrgykLOm0WTREgr_sNLVciXBXd6EQ-Zh2L7hd38noJm1P_MIr9_
EDRWAhoRLXPQ
```

The same, as a bearer token \(for an HTTP `Authorization` header\)

### Body 💪 <a id="body-"></a>

* `aud` "Audience" — the ID of who it's intended for \(the "to" field\)
* `iss` "Issuer" — ID of who sent it \(the "from" field\)
* `nbf` "Not Before" — Unix timestamp of when it becomes valid \(typically when it was created, but not always\)
* `exp` "Expiry" — Unix timestamp of when it stops being valid
* `scp` "Scope" — The scope of things it's able to change \(e.g. a file system path\)
* `ptc` "Potency" — what rights comes with the token \(in this case it's append only\)
* `prf` "Proof" — an optional nested token with equal or greater privileges

These are then all signed with the user's private key. This key must match the public key in the `iss` field \(user IDs are public keys\), directly authenticating the token. As the token is a complete description of access, this token is self-validating with no need to look at other data or services.

#### Delegation 🤝 <a id="delegation-"></a>

What if you want to grant another user or service the ability to perform some action on your behalf? As long as they have a valid UCAN, they can wrap it in another with equal or lesser rights and include the original in the `prf` field.

Since every UCAN layer is self-signed, we can trace back to the root \(no `prf` field\), and know who the delegate is acting as. This chain of tokens is itself is the proof that you're perform some action.

For example, here's a chain:

```text
"prf":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.
       eyJhdWQiOiJkaWQ6a2V5OnpTdEVacHpTTXRUdDlrMnZz
       emd2Q3dGNGZMUVFTeUExNVc1QVE0ejNBUjZCeDRlRko1
       Y3JKRmJ1R3hLbWJtYTQiLCJleHAiOjE1ODgyNjU0MjAs
       ImlzcyI6ImRpZDprZXk6ejFMQm53RWt0d1lMcnBQaHV3
       Rm93ZFZ3QUZYNXpwUm85cnJWendpUlJCQmlhQm9DUjdo
       TnFnc3RXN1ZNM1Q5YXVOYnFUbVFXNHZkSGI2MVJoVE1W
       Z0NwMUJUeHVhS1UzYW5Xb0VSRlhwdVp2ZkUzOWc4dTdI
       UzlCZUQxUUpOMWZYNlM4dnZza2FQaHhGa3dMdEdyNFpm
       ZmtVRTU3V1pwTldNNlU2QnFka3RaeG1LenhDODV6TjRG
       QzlXczVMSHVHZnhhQ3VCTGlVZkE3cUVZVlN6MVF1MXJa
       RHBENk55ZlVOckhKMUVyWmR1SnVuOTc2bmJGSHJtRG5V
       NDdSY1NNRkVTYk5LRGkxNDY3dFJmdWJzTXJEemViZENG
       S1EybTFBWXlzdG8yaTZXbWNudWNqdDN0bndUcWU3Qm84
       TDFnOGg4VUdBOTQ2REYzV2VHYlBVR3F6bnNVZExxNlhM
       Q21KekprSm1yTnllWmtzd2R5UFgyVnU2SjlFNEoxMlNi
       M2g5ZHM3YXRCeWFtZnRpdEVac2Y2aFBKa0xVWEdUaFlw
       Q25tUkFBclJSZlBZMkg2Y0tEYzdBY25GUHlOSEdrYWI1
       WkZvNHF2Z0JaeXRiSzFLNW9EM0hmUTZFMnliTGh5QzJi
       OGk1d282REx0bTl1Zml4U0pOTlRIN1Vpa2s4OENtZXJ0
       S1I3czEyQ0sxV0xFTTNadTVZQlpOcGhuamo3Y3A4UVRv
       ZEFlaFJQVjlORzFDTEVBTUpWTjc5RHZZZTZTZmlhZkpv
       YmN2ZkQ4bnBmUzZqY2VqY3lvdVFiRXBLREc3UUFuS1M0
       OFA0QXZnQnFEdmZOVWU1NGpNa2s2cjZDb1g0TGNZR0h1
       a1pERW5lYTlrd2tFb1hrVVlTNGoxQWZiS2g0NEZ6U3VY
       YlFxWm5qalZwVGh4Q05tbU5uMUU0cUhtc0ZrdkdvRjNG
       TjU1Q1Brb0dmREN2eVFKZ3Ftc0ZtcGVUSlN5OXd6djRN
       dmJxcHVBVHhyN2V5eHNHZUNXUWtjRHd1YjMyaW5HcFIz
       cmVUZnpSSkVDQ0ZaYXJuWGRjQzVQaWRha2IxV3U4TCIs
       Im5iZiI6MCwicHRjIjoiQVBQRU5EIiwicHJmIjpudWxs
       LCJzY3AiOiIvIn0.leyE9w2TF28espPq6mOWziQuJny2
       GHH_wajV6S9q4gF9SLP-i9JaX_XbkHlE1GhpQ36gSs6F
       v4_AXSuJzDkUhnAA-oPsI5bSHl28XbobzqdmXtQ2liK-
       Gum7kUtF1CPXlIamV0NIUlCKLlaUgFod5ZQvvA19kMHU
       ugDGm8O3G98TSm3qLlG-eoFNVXr0NSpvLeui3kQbdBsP
       GMykaTsUn1fNLI3oKkK6JvUIq4po6gIidTdOJDlS7y_W
       4bdMXUQcTprtpd2QmTqwTzws9tu4GBdx7q1vz35LiG39
       ohhRs2NKB4rxbZK2O9kX1G2xLMSETE_YT9GR04XWMnFo
       eIodsg"
```

Nested proof

You'll notice that the nested proof is encoded as a bearer token. This is because it needs to include its signature to prove that it's valid, and a JWT signature is on the content encoded this way.

This token is thus valid as long as:

* All token signatures are correct
* The time range, potency, and scope of `prf` are greater-or-equal to the enclosing token
* The outer token's `iss` field matches the `prf`'s `aud` field \(chain "to" and "from" correctly\)
* The timestamps are valid at the present time

#### Hashing ️ 🏎️ <a id="hashing-"></a>

These chains can get large, so you can optionally hash the outermost one before sending to a server. This acts as a "content address", meaning that if the service hasn't seen it before, it can separately request that token, but if it already has it in cache and doesn't need to get it over the network. Since hashes are much smaller than their content, this can save a lot of bandwidth on repeated requests.

```text
"prf": "QmU5WJTTp9vtMN1PBJpTV9xWXbTFBcWx3qjPGuXJXtujyd"
```

Same as the example above, but with the proof compressed to a content address

