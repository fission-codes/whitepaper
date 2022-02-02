# Table of contents

* [Abstract](README.md)
* [Contributors](contributors.md)

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
* [Command & Mutation](access-control/ucan.md)

## Accounts

* [Registration](accounts/verifiable-claims.md)
* [Delegation](accounts/device-linking.md)
* [Login](accounts/login/README.md)
  * [AWAKE](accounts/login/awake.md)
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
      * [I-Number](file-system/partitions/private-directories/concepts/i-number.md)
      * [Namefilter](file-system/partitions/private-directories/concepts/namefilter.md)
      * [Cipher Chunking](file-system/partitions/private-directories/concepts/cipher-chunking.md)
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
* [Distribution](file-system/distribution/README.md)
  * [DNSLink Backstop](file-system/distribution/backstop.md)
  * [Real Time](file-system/distribution/real-time.md)
  * [Ephemeral Sessions](file-system/distribution/ephemeral-sessions.md)

## Apps

* [Hosting](apps/hosting.md)
* [Layout](apps/layout.md)
* [Clone](apps/clone.md)
* [Follow](apps/follow.md)

## Database

* [WNKV](database/untitled-1.md)
* [WNDB](database/wndb.md)

## Dynamic FaaS

* [Distributed & Edge FaaS](dynamic-faas/distributed-functions-as-a-service.md)
