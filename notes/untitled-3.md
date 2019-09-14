# \*\*Notes to Self\*\*

Just like Rails met the needs and context of 2005, Asimov meets the needs and context of 2020, 2025, and beyond.

While our team absolutely appreciates code that is correct-by-construction, total, and other strong guarintees, we do not believe that the typical developer is interested in adopting radically new ways of working. The march of progress is slow, and while even simple conepts like types and FP are showing their influence, very few teams have fully-embracerd these long-standing techniques.

> Computer science is no more about computers than astronomy is about telescopes ~ Edsger Dijkstra
>
> The art of programming is the art of organizing complexity, of mastering multitude, and avoiding its bastard chaos as effectively as possible ~ Edsger Dijkstra

SCaaS

Note to self: DNS TXT records should come with a signature of the pointer





SCaaS

Notes to self

* DNS TXT records should come with a signature of the pointer
* Probably no need for gateways in any case
  * IPFS-in-browser
  * Ethereum primarily metatransactions and SCaaS





* Introduction
  * Setting the Stage
  * Core Assumptions
  * The End of History
  * Progressive Decentralization
  * Open Infrastructure
* High Level Architecture

-- Part I: Components --

* Unified Storage
  * Perspective
    * Features: nondestructive, censorship resistance as data integrity, CDN
  * Files
  * Database
  * PubSub
  * Data Security
  * Conjecture
  * Use
    * Static Assets
    * Abstact Models
    * Database rows, docs, &c
    * Package hosting
    * Event sourcing
    * "Host"
* Digital Scarcity
  * Perspective \*
  * Identity
  * ACLs
  * Assets
  * Implementation
    * Layer-2
    * Metatransactions
    * SCaaS, briefly
  * Use
    * ID & recoverable DID
    * Payment
    * Ownserhip
    * Authorization
    * Revokable ACLs
    * Autonomous code
    * Time Verifyablilty
* Portable Compute
  * Perspective \*
  * Wasm
  * Distributed Compute
  * Event Sourcing
  * SCaaS
  * Short intro to ID
  * Distributed FaaS
    * Trusted & Altruistic
    * Trustless
      * Infinite-Loop Economics \(unbounded gas\)
      * Autonomous code / bots
  * Use
    * Front end
    * Local compute
    * Seamless compute
    * FaaS
    * MPC

-- Part II: Cross Cutting --

* Applications
  * Perscective
  * UNIX-inspired File DAG
  * Code
* Identity
  * Perspecive
  * Encryption
    * Uses
    * Constraints \(e.g. triplesec\)
* Shared Mutable State
  * Perspective
    * "Deployment"
    * Data Updates
    * Eventual consistency
  * DNS Automation
* Interaction
  * Perspective \*
  * Actor Model
    * Message passing / Persistent mail boxes
  * Streams
    * Global partial order
    * Subgraph Merging
    * Subgraph Reorder

-- Conclusion --

* Conclusion

