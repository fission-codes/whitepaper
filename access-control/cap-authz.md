# Object Capability Model

Access control is mostly widely achieved via access control lists \(ACLs\): metadata or database tables that specify which user is allowed to perform which actions. This requires that the application boundary enclose all of the data and users. In fully centralized systems, this works reasonably well since all requests go through the same controlling source. However, as real world use cases inevitably expand, ACLs need to cover increasing complexity, exceptions, and roles. It often becomes error prone to write and maintain, and makes the ACL subsystem itself a target for attackers.

The object capability \(OCAP\) model works in the opposite mode. While ACLs are _reactive and centralized_, OCAP is _proactive and decentralized_ — an agent is allowed to perform some action if they have poof of those rights. This makes access control very granular, work offline, and in certain variants \(liek ours\) empowers users to delegate rights to others to act on their behalf. This greatly simplifies access control by presenting a document that includes what the user is permitted to do. The complexity arises from managing which credentials exist, and revoking them. The standard approch is to maintain a public revocation list, which is checked on each request. We will go into that in more depth later.

Fission uses a blend of correct-by-construction read auth and cryptographically-secured write certificates. This is a trustless method suitable to centralized, decentralized, peer-to-peer, and local-first applications. In essence, it means using a mix of signature chains to control who has access to what. Read access is granted by the mere fact that someone has the correct decryption key. Write access is mediated with signature chains, in a similar way to the common X.509 certificate. Rather than have a centralized application handle access control, this is usable everywhere, and under any circumstance.



