# Revocation

## Efficient Distributed Key Revocation

### One-Liner

Decentralized authorization token blocklist / 

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

