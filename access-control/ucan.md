# Command & Mutation

Fission uses UCANs for command and mutation auth. They are [JWT](https://blog.fission.codes/auth-without-backend/jwt.io)s that contain special keys. You can find their specification [here](https://github.com/ucan-wg/spec). UCAN was prioneered at Fission; as it became adopted by the wider community was spun out into a shared space with the various implementations from the organizations that have adopted it.

Commands and mutations are relegated via capability endowment from the account that initially created the resource (the ”root owner”). This is a form of delegated access, where an agent (identified by DID) that has been granted certain capabilities may redelegate a subset to another agent. One of the most common tasks for apps is authorizing users to perform some action, like storing new data to storage, updating records, or fetching a file.&#x20;

Traditional app architecture has many users share one database ("multi-tenant"), with all user data fully interleaved with each other. Authorization here is primarily focused on keeping users from editing each other's records on this shared infrastructure. The server's rules give fairly coarse-grained control. Due to the inevitable exceptions to these rules, the logic becomes increasingly complex over time.

Typically, in more distributed setups — such as microservices — all requests are funneled through a central authorization service. Over time this causes several challenges, including complex logic, cost of maintenance, tricky edge cases, and difficulty managing traffic spikes. In short: it doesn't scale well.

Even incumbents like[ Google are moving away from the traditional auth server model](https://research.google/pubs/pub41892/) to overcome the above challenges. Fission has different constraints from Google and Amazon, but can adapt a lot of these ideas for our purposes. Essentially they're moving from a central auth server setup to a distributed model where more power is delegated to services.

What if we learn from Google's approach (plus older approaches like [SDSI/SPKI](https://tools.ietf.org/html/rfc2693) and [X.509](https://en.wikipedia.org/wiki/X.509)) but took it to its logic conclusion? Meet the UCAN: self-signed tokens are a JWT variant of macaroons, SPKI, and OAuth 2. These are still bearer credentials, but paired with better public key infrastructure and distribution, self-signed with no need for specialized certificate authorities (”CA”). You can think of UCANs as essentially a decentralized certificate that can be used to access external services on behalf of yourself or another user that has granted you permission. They work at the edge, in the presence of network partitions, fully offline, in traditional cloud infrastructure, and so on.

![UCAN Sam](https://s3.fission.codes/2020/05/UCAN\_SAM-1.png)
