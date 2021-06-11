# Table of contents

* [Abstract](README.md)

## Introduction

* [Context](introduction/untitled-1.md)
* [The Constellation](introduction/the-constellation.md)
* [High Level Architecture](introduction/untitled-2.md)

## Foundation

* [Unified Storage](foundation/untitled-4.md)
* [Digital Scarcity](foundation/untitled-5.md)
* [Portable Compute](foundation/portable-compute.md)

## Authorization

* [Overview](authorization/id-overview.md)
* [Self-Certified](authorization/self-certified.md)
* [Challenge-Based](authorization/challenge.md)
* [DID Document](authorization/did-doc.md)

## Access Control

* [Object Capability Model](access-control/cap-authz.md)
* [Query Access](access-control/query-and-read.md)
* [Command & Mutation](access-control/ucan/README.md)
  * [Concepts](access-control/ucan/concepts.md)
  * [JWT Structure](access-control/ucan/jwt-authentication.md)
  * [Attenuation](access-control/ucan/webnative-attenuation/README.md)
    * [Account](access-control/ucan/webnative-attenuation/account.md)
    * [Domain Name](access-control/ucan/webnative-attenuation/domain-name.md)
    * [File System](access-control/ucan/webnative-attenuation/file-system.md)
    * [Web Application](access-control/ucan/webnative-attenuation/app.md)
  * [Revocation](access-control/ucan/revocation.md)
  * [Differences from OAuth](access-control/ucan/differences-from-oauth.md)

## Accounts

* [Registration](accounts/verifiable-claims.md)
* [Delegation](accounts/device-linking.md)
* [Login](accounts/login/README.md)
  * [AWAKE](accounts/login/quake.md)
* [Recovery](accounts/recovery.md)
* [Multifactor Authentication](accounts/mfa.md)

## File System

* [Basics](file-system/file-system-basics/README.md)
  * [Layers](file-system/file-system-basics/layers.md)
  * [Virtual Nodes](file-system/file-system-basics/vnodes.md)
  * [Reduction Caches](file-system/file-system-basics/reduction-caches.md)
  * [Versioning](file-system/file-system-basics/versioning.md)
  * [Events](file-system/file-system-basics/journal.md)
* [Partitions](file-system/partitions/README.md)
  * [Root](file-system/partitions/root.md)
  * [Public](file-system/partitions/public-directory.md)
  * [Pretty](file-system/partitions/pretty.md)
  * [Private](file-system/partitions/private-directories/README.md)
    * [Concepts](file-system/partitions/private-directories/concepts/README.md)
      * [CryptDAG](file-system/partitions/private-directories/concepts/cryptree.md)
      * [Spiral Ratchet](file-system/partitions/private-directories/concepts/spiral-ratchet.md)
      * [Namefilter / i-number](file-system/partitions/private-directories/concepts/namefilter.md)
    * [Data Layer](file-system/partitions/private-directories/data-layer.md)
    * [File Layer](file-system/partitions/private-directories/platform-layer.md)
    * [Append](file-system/partitions/private-directories/append.md)
  * [Revoked](file-system/partitions/revoked.md)
* [Unlisted](file-system/unlisted-directories.md)
* [Share](file-system/shared.md)
* [Links](file-system/links.md)
* [Events](file-system/events.md)
* [Consistency](file-system/consistency/README.md)
  * [Synchronization](file-system/consistency/synchronization.md)
  * [Transactions](file-system/consistency/transactions.md)
  * [Retroactivity](file-system/consistency/retroactivity.md)
  * [Fault Tolerance](file-system/consistency/fault-tolerance.md)
* [Distribution](file-system/distribution/README.md)
  * [Backstop](file-system/distribution/backstop.md)
  * [Real Time](file-system/distribution/real-time.md)
  * [Performance](file-system/distribution/performance.md)

---

* [App Hosting](app-hosting.md)

## Apps

* [Hosting](apps/hosting.md)
* [Layout](apps/layout.md)
* [Clone](apps/clone.md)
* [Follow](apps/follow.md)

## Key-Value

* [WNKV](key-value/untitled-1.md)

## Database

* [WNDB](database/wndb.md)

## Dynamic FaaS

* [Distributed & Edge FaaS](dynamic-faas/distributed-functions-as-a-service.md)

