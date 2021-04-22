# High Level Architecture

The Fission architecture is made up of six modular components, which group loosely into three segments:

1. Unified Storage

   a. Files

   b. Database

2. Digital Scarcity

   a. Identity

   b. Assets

3. Portable Compute

   a. Private & Trusted

   b. Public & Trustless

```text
               +--------------------------------------------------------------------------------------------------------+
               |                        +---------------+                                                               |
+-----+        | Unified Storage        | Files (IPFS)  |                                                               |
|     |        | ===============        +---------------+                                                               |
|     | -----> |                                                                   +------------+     +------------+    |
|     |        |                        +--------------------+                     |            |     |            |    |
|     |        |                        | Database (OrbitDB) |                     |            |     |            |    |
|     |        |                        +--------------------+                     |            |     |            |    |
|Route|        +-------------------------------------------------------------------+            +-----+            +----+
|(DNS)|                                                                            |            |     |            |
|     |        +-------------------------------------------------------------------+            +-----+            +----+
|     |        |                         +----------------------------------+      |            |     |            |    |
|     |        | Digital Scarcity        |Identity (Public Key Cryptography)|      |            |     |            |    |
|     |        | ================        +----------------------------------+      |            |     |            |    |
|     | <----> |                                                                   | Encryption |     |    FaaS    |    |
|     |        |                         +----------------------------------+      |            |     |            |    |
+-----+        |                         |Assets (primarily ERCs 20 and 721)|      |            |     |            |    |
               |                         +----------------------------------+      |            |     |            |    |
               +-------------------------------------------------------------------+            +-----+            +----+
                                                                                   |            |     |            |
               +-------------------------------------------------------------------+            +-----+            +----+
               |                        +-------------------------------+          |            |     |            |    |
               | Portable Compute       | Private / Trusted (Wasm & JS) |          |            |     |            |    |
               | ================       +-------------------------------+          |            |     |            |    |
               |                                                                   +------------+     +------------+    |
               |                        +------------------------------------+                                          |
               |                        | Public / Trustless (Ethereum & L2) |                                          |
               |                        +------------------------------------+                                          |
               +--------------------------------------------------------------------------------------------------------+
```

Cutting across these are two second-order modules that rely on the three core areas:

1. Encryption
2. Functions-as-a-Service

The vast majority of end users will not have cryptographically-backed routing available locally for some time. To bridge this gap, Fission leverages public DNS infrastructure as a backward-compatibility bridge.

