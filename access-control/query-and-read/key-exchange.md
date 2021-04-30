# Sharing

Granting another user, machine, or browser tab read/query access to some resource \(e.g. WNFS\) is as simple as handing them a copy of the key and a pointer to the relevant data.

This is self-contained information, and transport agnostic — any secure channel works. In Fission’s system, this is generally handled directly in WNFS via the `shared_by_me` and `shared_with_me` segments. We also transfer data via a bootstrapped secure channel on top of a WebRTC-based pubsub, and even directly in query parameters.

{% hint style="danger" %}
Note that anyone with read access can also grant the same access to others without alerting the owner. Revocation exists in the current spec \(next section\), but methods for more granular access control exist, though they have tradeoffs in availability, time, storage, or all three. We believe that in practice this is not required for the 98% use case, and that key rotation is sufficient. Different protocols can be implemented on top of this layer if needed.
{% endhint %}



