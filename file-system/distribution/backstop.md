# DNSLink Backstop

The DNSLink acts as the primary means of distributing changes to a WNFS, though it is by no means the only method of doing so. As a slow by durable pointer with wide reach, DNSLink behaves as a rough proxy for where the swarm will have advanced to. Bootstrapping fresh history via DNSLink is very fast. Being a distributed system, there are always be partitions \(even if just latency\), so this is not a single source of truth, but does always make progress.

