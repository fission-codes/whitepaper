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
  * [Anatomy](file-system/file-system-basics/anatomy.md)
  * [Versioning](file-system/file-system-basics/versioning.md)
  * [Head Synchronization](file-system/file-system-basics/head-synchronization.md)
  * [Journal](file-system/file-system-basics/journal.md)
* [Sections](file-system/sections/README.md)
  * [Root](file-system/sections/root.md)
  * [Public](file-system/sections/public-directory.md)
  * [Pretty](file-system/sections/pretty.md)
  * [Private](file-system/sections/private-directories/README.md)
    * [Protocol Structure](file-system/sections/private-directories/protocol-layer.md)
    * [Platform Structure](file-system/sections/private-directories/platform-layer.md)
    * [Read Access](file-system/sections/private-directories/read-access.md)
    * [Append Access](file-system/sections/private-directories/append-access.md)
  * [Unlisted](file-system/sections/unlisted-directories.md)
  * [Share](file-system/sections/shared.md)
* [Consistency](file-system/consistency/README.md)
  * [Transactions](file-system/consistency/transactions-1.md)
  * [Event Sourcing](file-system/consistency/event-sourcing.md)
  * [Transactions](file-system/consistency/transactions.md)
* [Distribution](file-system/distribution/README.md)
  * [Backstop](file-system/distribution/backstop.md)
  * [Performance](file-system/distribution/performance.md)
  * [App Hosting](file-system/distribution/app-hosting.md)
* [Multiuser](file-system/multiuser.md)

## Database

* [Untitled](database/untitled.md)

## Dynamic FaaS

* [Distributed & Edge FaaS](dynamic-faas/distributed-functions-as-a-service.md)

## Conclusion

* [Recapitulation](conclusion/recapitulation.md)
* [Afterward](conclusion/afterward.md)

