# Command & Mutation

Commands and mutations and relegated via capability endowment from the account that initially created the resource \(the ”root owner”\). This is a form of delegated acess, where an agent \(indexed by DID\) that has been granted certain capabilities may redelegate a subset to another agent.

While Fission leverages self-sovereign identity, it still needs to be able to interact with existing systems and flows with the minimum amount of reworking possible. Bridging delegated authorization such as tokens \(commonly used by Web 2.0 systems\) is fairly straightforward, and only minor modifications for the service, and continues to make sense in a web3 context.

Fission's self-signed tokens are a JWT variant of [macaroons](https://research.google/pubs/pub41892.pdf) and SDSI/[SPKI](https://tools.ietf.org/html/rfc2693). These are bearer credentials. You can think of them as essentially a decentralized certificate that can be used to access external services on behalf of another user.

