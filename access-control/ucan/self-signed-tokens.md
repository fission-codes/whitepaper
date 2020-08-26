# Self-Signed Authorization

While Fission leverages self-sovereign identity, it still needs to be able to interact with existing systems and flows with the minimum amount of reworking possible. Bridging delegated authorization such as tokens \(commonly used by Web 2.0 systems\) is fairly straightforward, and only minor modifications for the service, and continues to make sense in a web3 context.

Fission's self-signed tokens are a JWT-implementation of [macaroons](https://research.google/pubs/pub41892.pdf). These are bearer credentials — essentially a decentralized certificate that can be used to access external services on behalf of another user.

## Token Format

## Subdelegation

 of the token to trusted third-parties with even more limited access.

## One-Time Tokens

There are times when tokens should be single use. This is generally in situations where multiple users need to atomically collaborate on some action via some aggresating service.

### Additional Requirements

* The token must include the recipient \(i.e. scoped to the receiver\)
* Each token must be unique
  * If a request needs to be made multiple times, adjust the nonce
* Either
  * Strictly enforce ordering with a monotonically-increasing nonce
    * Version 1: If a require every token to be received before allowing the next
    * Version 2: Accept any token, but invalidate any previous tokens
  * Keep a rolling tally of used token by hash
    * Drop expired tokens from this cache since they can no longer be replayed

### Pros

* Secure & authenticated
* Self-sovereign
* Can Pre-signable
* Not susceptible to replay attacks
* Can be used as an authenticated proof of intent

### Cons

* Require the user \(or an authorized delegate\) to be online to sign a token
* The end-recipient maintain a token invalidation list for replay resistance to hold
* Typically requires the entire payload to be signed
  * i.e. requires all of the optional claims in the [JWT Authentication](https://app.gitbook.com/@runfission/s/whitepaper/~/-Lyqf_PlC7NGcLgfnH4p/identity/jwt-authentication#claims) section



