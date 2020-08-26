# JWT Structure

Why JWT?

From a purely technical perspective, there’s nothing special about the JWT. Socially, the JWT is currently the most common and widely understood format for tokens in web applications today.



`rsc` \("resource"\) is a generalization of the UCAN v0.1.0 `scp` \("scope"\) parameter. The TL;DR is it no longer only refers to the filesystem, but also apps and domains. It could be used for more in the future, as well.

We're dropping support for 0.1.0, since it was effectively unreleased and this covers more use cases that we're using imminently.

## TL;DR

* Drop `scp` field
* Add `rsc` field

## v0.2.0 Claims Schema

```typescript
{ 
  "iss": DID,
  "aud": DID,

  "rsc": ScopedResource,
  "ptc": Potency,
  "prf": Proof,

  "exp": UTCTime,
  "nbf": UTCTime
}
```

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

This may change _slightly_ this morning, but wanted to catch y'all up on what's on the branch now and design goals

## Existing

The existing v0.2.0 UCAN spec _only_ supports FFS

There's a `scp` \("scope"\) field that contains a file path, which is represented as a string

## Updated

### Design goals

* Support delegated access to FFS, apps, and domains
* Open to extension to more \(arbitrary\) resources
* Not explicitly reveal \(i.e. list out\) _all_ of the resources that you have access to
* Remain as compact as possible

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

There is no list of resources. If there were, then token delegation starts to reveal a bunch of information about which resources are available to someone higher in the chain. i.e. if I'm trying to delegate access to `A` , but two layers up it talks about `[A,B,C]` , then I'm revealing information about me having access to `B` and `C` . This information is irrelevant to the current case, takes up extra characters, and potentially awkward \("I don't want to reveal that Boris has access to X, but my access to Y has that listed sooooo"\)

Saying just "everything" solves for our delegation case, is compact, and doesn't actually transmit _information_

But maybe I'm wrong and we want to list multiple resources?

This design also means that you'll need to hold onto multiple UCANs. This was already the case, but as a concrete case: if we're in a team and working on 5 apps together, you'll have 5 UCANs. If you link machines, you'll have one UCAN for "everything".

My biggest open question in this design is if we need the layer of nesting under `rsc` , or if it should just be directly in the top level \(merged directly into the claims\). They're isomorophic, but this to me feels clearer about the intent \(human factors\)



This was going in a post for Imperial College, but writing out the problem lead to a series of solutions. Sooo moving here.

Copy/paste from Discord

> Oh god I'm solving problems while I'm typing up tasks for IC :stuck\_out\_tongue: This is what happens when I have time to think about stuff :laughing: So, UCAN revocation We only need to keep a list of non-expired, invalid tokens UCANs are content addressable, as are the PKs involved So... keep a list of bad CIDs in FLOOFS to start at least Clear the cache from time to time Keep this cache at a well-known path Potentially even include the actual bad UCAN Service providers use a Merkel proof to know that nothing has changed since they last looked Crap, that could actually work :woman\_shrugging: Removing it from the IC write up Moving to a new Talk post This... is a problem with PKI that has existed since the 80s Vertically integrated tech stacks FTW

## Efficient Distributed Key Revocation ⛔

### One-Liner

Decentralized authorization token blocklist

### Background

* Public key infrastructure without a blockchain
* Self-sovereign identity, where users can create keys as needed
* Sybil resistance is _not_ a design goal \(in fact, likely an anti-goal\)
* We keep the user's "root" PK in a DNS `TXT` record
  * But really pick your favourite distribution method
* They then create & sign tokens delegating a subset of their rights
  * Other machines, apps, or other users
  * Here's a [\(very\) high level overview](https://blog.fission.codes/auth-without-backend/)
* This delegation is signed — in layers — by the chain going back to the root user
  * A mix of macaroons and SPKI/SDSI auth
  * This doubles as authentication for these authorization tokens
* These delegations are typically time limited \(unless you're linking machines as "root"\)
  * But this time can be arbitrarily in the future

### Scenario

* The user discovers abuse of a delegation \(e.g. stolen laptop, memory hack, etc\)
* They want to revoke that key, and thus also any sub-delegations
* The sub-delegations may be used anywhere
* The signatures are still valid, but the token should not be

### Discussion / Sketch

We're using a proactive authorization model for mutations \(OCAP\). This works very well offline and fully P2P, takes our servers out of the equation completely, and puts all the power in the user's hands. Today these authorization tokens are fully self-contained. However, the downside is what to do when you've found misuse of these tokens, it would be good to be able to revoke them.

A naive approach is token "generations" — essentially version the tokens and publish the latest generation somewhere. Everyone should disallow anything below the current generation. We don't want to rotate _all_ of these tokens if _one_ is bad. In fact, it may not even be possible \(or at least very messy\) if you no longer have access to the original root key, only keys with full delegated access.

The good news is that we only need to worry about potentially revoking tokens haven't expired. Proactive auth means that these tokens _should_ be short lived, but that's not enforceable.

The solution should be fully decentralized, not depend on our servers, be quick to look up, and not require downloading oodles of data.

\[Having a brain wave, moving to Discord\]





To guard against replay attacks, all authenticated requests to and from Fission are one-time use only, and feature a unique nonce inside a sliding time window.

Fission uses the familiar JWT format, plus a few additional keys.



[Fission](https://blog.fission.codes/)

* [WEEKLY LINK ROUNDUP](https://blog.fission.codes/tag/fragments/)
* [EVENTS](https://blog.fission.codes/tag/event/)
* [PRESENTATIONS](https://blog.fission.codes/tag/presentation/)
* [HOMEPAGE ⇗](https://fission.codes/)
* [FORUM ⇗](https://talk.fission.codes/)

[Subscribe](https://blog.fission.codes/auth-without-backend/#subscribe)7 MAY 2020/[TECHNOLOGY HIGHLIGHT](https://blog.fission.codes/tag/technology-highlight/)

## UCAN: Authorizing Users Without a Back End

![UCAN: Authorizing Users Without a Back End](https://s3.fission.codes/2020/05/zdenek-machacek-EtxsgEcHnZg-unsplash.jpg)

Fission is building a system which "makes the right thing the easy thing." It lets you write apps for the browser without having to write or deploy a back end. We're making use of fairly recent browser features and W3C standards to make this all possible. Read on for a technical summary, or [join us in the developer forum](https://talk.fission.codes/) to get into more detail.

One of the most common tasks for apps is authorizing users to perform some action, like storing new data to storage, updating records, or fetching a file. 

Traditional app architecture has many users share one database \("multi-tenant"\), with all user data fully interleaved with each other. Authorization here is primarily focused on keeping users from editing each other's records on this shared infrastructure. The server's rules give fairly coarse-grained control. Due to the inevitable exceptions to these rules, the logic becomes increasingly complex over time.

Even in a microservice architecture, typically all requests are funneled through a central authorization service. Over time this causes several challenges, including complex logic, cost of maintenance, tricky edge cases, and difficulty managing traffic spikes. In short: it doesn't scale well.

Even incumbents like[ Google are moving away from the traditional auth server model](https://research.google/pubs/pub41892/)to overcome the above challenges. Fission has different constraints from Google and Amazon, but can adapt a lot of these ideas for our purposes. Essentially they're moving from a central auth server setup to a distributed model where more power is delegated to services.

What if we learn from Google's approach \(plus older approaches like [SDSI/SPKI](https://tools.ietf.org/html/rfc2693)\) but took it to its logic conclusion?

###  <a id="introducing-ucans"></a>

At a high level, User Controlled Authorization Networks \(UCANs\) are a way of doing authorization \("what _you can_ do"\) where users are fully in control. There's no all-powerful authorization server, or server of any kind required. Everything that a users is allowed to do is captured directly in a key or token, and can be sent to anyone that knows how to interpret this format.

Since all Fission accounts are equipped with a global ID and cryptographic keys, we were able to design a system that has very few assumptions and thus works in a huge number of situations.

This setup has several advantages:

1. Low effort: developers don't need to write and maintain complex access logic
2. Familiar: uses very common JSON Web Tokens \(JWTs\)
3. Invisible: users don't need to know that anything special is happening
4. Flexible: access can be granted as coarse or granular as the end users wants
5. Scalable: no auth server bottleneck / scales infinitely
6. Secure: military-grade encryption
7. Collaborative: users and services and delegate a subset of their access to others
8. Self-contained: the token contains all the information needed to verify it

UCANs are all that we need to sign into multiple machines, delegate access for service providers to do things while we're offline, securely collaborate on documents with a team, and more. We get the flexibility of fine- or coarse-grained control, all controlled by the one who cares about the data the most: the user.

We've implemented this as the authorization system for Fission, and are also making this available as a building block for developers to solve user authorization and delegation within their own applications.

There are some actions that a user needs the help of another user or service to perform. For example: sending an email, or updating DNS.

In a traditional OAuth based system, the "account" lives entirely on the server, and the user is granted access with a token_._ In Fission's design, the account is a key pair, and a UCAN is equivalent to an OAuth token. OAuth is designed for a centralized client/server world. UCANs are the distributed user controlled equivalent.

UCANs are simply [JWT](https://blog.fission.codes/auth-without-backend/jwt.io)s that contain special keys. Much of this will look familiar if you've done web auth in the past decade or so. Here's an example:

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

Let's break that down:

### Header 📋 <a id="header-"></a>

This is a standard JWT header, plus a `uav` field.

* `alg` — type of signature
* `typ` — state that this is a JWT
* `uav` — "UCAN version" \(so we can track the format of when it was issued\)

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

## Conclusion <a id="conclusion"></a>

UCANs are a straightforward way of doing authorization that leverage the public key infrastructure already baked into Fission. This is essentially authorization-at-the-edge with familiar JWTs. Since the token is self-contained, it's infinitely scalable. It's also very flexible: the user can grant root access to everything, or grant a tab write access to a single object for one minute.

## Complete Example

NOTE THIS SECTION IS OUT OF DATE AND BEING _HEAVILY REWORKED 👷‍♀️🚨_ 

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



