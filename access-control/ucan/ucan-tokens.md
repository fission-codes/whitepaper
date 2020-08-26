# UCAN Tokens

At a high level, UCANs \(“User Controlled Authorization Network”\) are a way of doing authorization \("what _you can_ do"\) where users are fully in control. There's no all-powerful authorization server, or server of any kind required. Everything that a users is allowed to do is captured directly in a key or token, and can be sent to anyone that knows how to interpret this format.

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

UCANs are simply [JWT](https://blog.fission.codes/auth-without-backend/jwt.io)s that contain special keys.

### User Controlled

Users control the data \(and other resources\) that they create. They also have the ability to delegate control to others.

### Authorization Network

“Network” here is meant in the sense of a graph.

The UCAN format is designed as an _authenticated_ digraph in some larger _authorization_ space. The other way to view this is as a function from a set of authorizations \(“UCAN proofs“\) to a subset output \(“UCAN capabilities”\).

![UCAN Sam](https://s3.fission.codes/2020/05/UCAN_SAM-1.png)

