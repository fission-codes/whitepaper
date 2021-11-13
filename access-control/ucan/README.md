# Command & Mutation

Commands and mutations and relegated via capability endowment from the account that initially created the resource (the ”root owner”). This is a form of delegated access, where an agent (identified by DID) that has been granted certain capabilities may redelegate a subset to another agent. One of the most common tasks for apps is authorizing users to perform some action, like storing new data to storage, updating records, or fetching a file.&#x20;

Traditional app architecture has many users share one database ("multi-tenant"), with all user data fully interleaved with each other. Authorization here is primarily focused on keeping users from editing each other's records on this shared infrastructure. The server's rules give fairly coarse-grained control. Due to the inevitable exceptions to these rules, the logic becomes increasingly complex over time.

Typically, in more distributed setups — such as microservices — all requests are funneled through a central authorization service. Over time this causes several challenges, including complex logic, cost of maintenance, tricky edge cases, and difficulty managing traffic spikes. In short: it doesn't scale well.

Even incumbents like[ Google are moving away from the traditional auth server model](https://research.google/pubs/pub41892/) to overcome the above challenges. Fission has different constraints from Google and Amazon, but can adapt a lot of these ideas for our purposes. Essentially they're moving from a central auth server setup to a distributed model where more power is delegated to services.

What if we learn from Google's approach (plus older approaches like [SDSI/SPKI](https://tools.ietf.org/html/rfc2693) and [X.509](https://en.wikipedia.org/wiki/X.509)) but took it to its logic conclusion? Meet the UCAN: self-signed tokens are a JWT variant of macaroons, SPKI, and OAuth 2. These are still bearer credentials, but paired with better public key infrastructure and distribution, self-signed with no need for specialized certificate authorities (”CA”). You can think of UCANs as essentially a decentralized certificate that can be used to access external services on behalf of yourself or another user that has granted you permission. They work at the edge, in the presence of network partitions, fully offline, in traditional cloud infrastructure, and so on.

## Overview

UCANs are simply [JWT](https://blog.fission.codes/auth-without-backend/jwt.io)s that contain special keys.

At a high level, UCANs (“User Controlled Authorization Network”) are a way of doing authorization ("what _you can_ do") where users are fully in control. There's no all-powerful authorization server, or server of any kind required. Everything that a users is allowed to do is captured directly in a key or token, and can be sent to anyone that knows how to interpret this format.

In a traditional OAuth based system, the "account" lives entirely on the server, and the user is granted access with a token_. _In Fission's design, the account is a key pair, and a UCAN is equivalent to an OAuth token. OAuth is designed for a centralized client/server world. UCANs are the distributed user controlled equivalent.

Because these typically originate with the user, the model is completely flipped from how we normally think about control. This DID exists _before _the user registers with a service. This should not seem too unnatural; the human controlling this DID existed before their account, too! Service are renting some of their namespace to this DID agent.

Since all Fission accounts are equipped with a _truly global_ ID and cryptographic keys, we were able to design a system that has very few assumptions and thus works in a huge number of situations. It is also extensible to other DID methods (please see the account section for more).

This setup has several advantages:

1. Low effort: developers don't need to write and maintain complex access logic
2. Familiar: uses very common JSON Web Tokens (JWTs)
3. Invisible: users don't need to know that anything special is happening
4. Flexible: access can be granted as coarse or granular as the end users wants
5. Scalable: no auth server bottleneck / scales infinitely
6. Secure: military-grade encryption
7. Collaborative: users and services and delegate a subset of their access to others
8. Self-contained: the token contains all the information needed to verify it

UCANs are all that we need to sign into multiple machines, delegate access for service providers to do things while we're offline, securely collaborate on documents with a team, and more. This provides flexible fine- or coarse-grained access policies, all controlled by the one who cares about the data the most: the user.

We've implemented this as the base authorization system for Fission. This is extensible and available as a building block for developers to solve user authorization and delegation within their own applications.

There are some actions that a user needs the help of another agent or service to perform, since they are not available directly in a browser. For example: sending an email, or updating DNS.

## Name

### User Controlled

Users control the data (and other resources) that they create. They also have the ability to delegate control to others.

### Authorization Network

“Network” here is meant in the sense of a graph.

The UCAN format is designed as an _authenticated _digraph in some larger _authorization_ space. The other way to view this is as a function from a set of authorizations (“UCAN proofs“) to a subset output (“UCAN capabilities”).

![UCAN Sam](https://s3.fission.codes/2020/05/UCAN\_SAM-1.png)
