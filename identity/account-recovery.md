# Account Recovery

A user must be able to recovery their account and file system in a privacy-preserving way that reveals no information to Fission.  
- **Read Access:** By decrypting a root AES key that reveals access to the `/private` branch of the filesystem   
- **Write Access:** By delegating full write access to a new DID through a UCAN

#### Constraints

* Users should have a set of recovery codes \(similar to 2FA recovery codes\), such that any one recovery code has the capability of restoring full access to a user's filesystem
* Upon recovery, a user's DID should have an unbroken chain of delegation from their original DID \(even though they no longer have access to that keypair\)
* Fission should not be able to restore or have access to a user's filesystem without access to their recovery codes
* Fission should have the ability to stop or slow access to a filesystem in the case of suspicious activity

Basic outline:

#### C**reation**

* Alice
  * generates 10 random codes \(of whatever length, let's say 10 hex chars\) 
  * hashes each of these codes _once_ to create 10 BLS secret keys \(`SK_a`\)
  * hashes each of these secret keys _again_
  *  sends the hashes to the server
* The server
  * generates 10 BLS secret keys \(`SK_f`\), one for each hash, and stores them in a database alongside the hashes
  * signs some arbitrary piece of data `challenge`,with each key
  * sends Alice a pair for each key of: `(sig_f, PK_f)` 
* For each key, Alice
  * signs the same arbitrary piece of data with each key and combines that signature with the relevant signature
  * combines the public key with the relevant public key from the server, thus determining a pair of `(sig_agg, PK_agg)`
  * determines a did for each `PK_agg`: `did:key:zAliceRecovery`
  * delegates a full permission UCAN \(`UCAN_recovery`\) to each `did:key:zAliceRecovery`, attested by her root  `did:key:zAlice` 
  * creates an `AccessFile` for each `UCAN_recovery` and includes the root AES key \(`R_root`\) to decrypt the user's private filesystem

```text
{
  root: did:key:zAlice,
  username: alice.fission.name,
  rootAesReadKey: RK,
  recoveryPartnerBlsPublicKey: PK_f,
  delegatedUcan: UCAN_recovery,
}
```

* Alice 
  * hashes `sig_agg` to determine an AES256 key `R_recovery` 
  *  encrypts `AccessFile` with each `R_recovery` key and stores it in their filesystem at `/recovery/{sha256(R_recovery)}`

#### **Recovery**

* The user 
  * enters one of their recovery codes
  * hashes the recovery code to reveal `SK_a` 
  * creates a new keypair `(SK_r, PK_r)` and associated DID `did:key:zAliceNew`
  * sends a request to the server including `hash(SK_a)` and `did:key:zAliceNew`
* The server 
  * looks up the relevant key to `SK_a` in the database: `SK_f` 
    * Checks that the key has not been used for recovery yet \(one time only\)
    * We can add a time delay on this part for added security. If a user reports their device missing or their security breached, this is also where we can halt an attacker.
  * signs the original `challenge` with `SK_f` to botain `sig_f`
  * signs a full permissioned UCAN \(`UCAN_new_f`\) from `did:key:zAliceRecovery` for `did:key:zAliceNew` 
  * sends `sig_f` and `UCAN_new` to Alice
  * marks that this keypair has been used in the DB
* Alice
  * signs a full permission UCAN \(`UCAN_new_a`\) from `did:key:zAliceRecovery` for `did:key:zAliceNew` 
  * combines `UCAN_new_a` with `UCAN_new_f` to create `UCAN_new`, a fully permission UCAN for `did:key:zAliceNew`
  * signs `challenge` with `SK_a` and combines the result with `sig_f` to obtain `sig_agg` 
  * hashes `sig_agg` to obtain  AES256 key `R_recovery` 
  * retrieves the encrypted `AccessFile` from `/recovery/{sha256(R_recovery)}` 
  * decrypts `AccessFile` with `R_recovery` 

## Account Reconstruction

In the event that all a user is locked out of all of their devices and loses all of their recovery codes, they can undergo account reconstruction. They are able to retain their username, namespace, and public filesystem. However, they are not able to gain access to their private filesystem, and account reconstruction resets all of the user's UCANs and then revokes access to any app that a user has permissioned.

To reconstruct an account, a user generates a completely new root AES key for their filesystem, as well as a new base keypair and related DID for their account. Fission sets their DID at `_did.${username}.fission.name}` to the newly generated DID.

The user then backs up their existing file system \(in the event that they do find the necessary recovery codes\), creates a new `/private` branch of the filesystem \(encrypted with the newly created root AES key\), and updates `_dataroot.${username}.fission.name` to the root of this new filesystem.

