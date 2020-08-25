# Sharing Query Access

Granting another user, machine, or browser tab read/query access to some resource \(e.g. WNFS\) is as simple as handing them a copy of the key and a pointer to the relevant data.

This is self-contained information, and transport agnostic — any secure channel works. In Fission’s system, this is generally handled directly in WNFS via the `shared_by_me` and `shared_with_me` segments. We also transfer data via a bootstrapped secure channel on top of a WebRTC-based pubsub, and even directly enrypted query parameters.

