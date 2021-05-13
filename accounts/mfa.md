# Multifactor Authentication

Multifactor authentication is possible entirely from within the UCAN construction, or via an external provider. This is very close to the common MFA systems seen today: a second party vouches for the validity of a login or other statement. This is mediated by a challenge, via email, SMS, or authenticator application. Once validated, this external party cosigns a UCAN, or aggregate decryption for read access.

### Cosigning

It is also possible for this method to be self-sovereign, where the end user manages their own m-of-n scheme.

### Countersigning

With countersigning, the outmost UCAN must provide proofs that meet some criterea. One such example is having a set of well-known DIDs that must have delegated valid rights to the action. This is especially helpful when a resource is shared among multiple users that don't have access to pairing-based cryptography.

