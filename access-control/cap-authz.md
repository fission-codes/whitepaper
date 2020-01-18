# Capability-Based Authorization

Access control has traditionally been managed by access control lists \(ACLs\): metadata or database tables that specify which user is allowed to perform which actions. This requires that the application boundary enclose all of the data. In fully centralized systems, this works reasonably well — though it's very error prone to write and maintain, and makes the application itself a target for attackers.

Fission uses correct-by-construction \(AKA "capability-based"\) authorization. This is a trustless method suitable to centralized, decentralized, peer-to-peer, and local-first applications. In essence, it means using raw cryptography to control who has access to what. Rights are granted by the mere fact that someone has the correct decryption key. Rather than have a centralized application handle access control, this is usable everywhere, and under any circumstance.

Fission does not currently link identities, but rather grants shared access control over resources.



