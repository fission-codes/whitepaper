# Device Linking

For convenience, a user must be able to control as many of their resources as possible without leaving the device. Since browsers are hostile places, this requires a trusted setup via a well-known authentication page. This helps guard against phishing and acts as a capability store.

Devices are "linked" by delegating strong rights to the new device. In the typical case, this is access to everything that the authorization-producing device has access to. This typically means transferring any top-level symmetric keys, and signing a maximally capable UCAN.

This transfer is highly sensitive. Luckily UCANs provide a correct-by-construction way of proving access: they can sign a non-delegating UCAN targeted at the requesting device \(see specification of the QUAKE protocol in the next section\).

