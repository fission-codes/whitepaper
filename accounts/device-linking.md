# Delegation

Authorization in an OCAP setting is extremely flexible and _uniform_. There is little difference in the mechanism for sharing a file vs logging into an app, though the actual capabilities being transferred will vary.

## App Delegation

Every app has its own unique DID. This is globally unique and nonexportable by default, which is to say that for the same person, the DID of the app at a particular URL will differ from their laptop to their phone. The app loads, a new agent is generated, and the user delegates some subset of their rights to this particular app agent.

## Device Linking

For app linking to be convenient, a user must be able to control as many of their resources as possible without leaving the device. Since browsers are hostile places, this requires a trusted setup via a well-known authentication page. This helps guard against phishing and acts as a capability store.

Devices are "linked" by delegating strong \(usually all\) rights to the new device. This typically means transferring any top-level symmetric keys, and signing a maximally capable UCAN. Wildcard capabilities allow delegation of resources that don't exist yet.

This transfer is highly sensitive. UCANs provide a correct-by-construction way of proving access: they can sign a non-delegating UCAN targeted at the requesting device \(see specification of the AWAKE protocol elsewhere in this paper\).

## Inter-User Capability Sharing

Delegating to another _user_ is the exact same mechanism as delegating to an app: send a UCAN and keys scoped to the desired resources. This can happen live \(via a protocol such as AWAKE\), but is much more convenient to be executed asynchronously via the sharing segment of WNFS.

