# Capability-Based Authorization

Access control has traditionally been managed by access control lists \(ACLs\): metadata or database tables that specify which user is allowed to perform which actions. This requires that the application boundary enclose all of the data. In fully centralized systems, this works reasonably well — though it's very error prone to write and maintain, and makes the application itself a target for attackers.

Fission uses correct-by-construction \(AKA "capability-based"\) authorization. This is a trustless method suitable to centralized, decentralized, and local applications. In essence, this means using cryptography to control who has access to what. Rather than have a centralized application handle access control, the rights are baked directly into keys, usable everywhere, with any application, online and offline.



