# Capability-Based Authorization

Access control has trditionally been managed by access control lists \(ACLs\): metadata or database tables that specify which user is allowed to perform whihc actions. This requires that the application boundary enclose all of the data. In fully centralized systems, this works reasonably well — though it's very error prone to write and maintain, and makes the application itself a target for attackers.

Fission uses correct-by-construction or "capability-based" authorization.

