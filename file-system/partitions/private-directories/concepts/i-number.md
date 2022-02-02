---
description: AKA i-nonce
---

# I-Number

The identity of a file is a _random_ 256-bit nonce. This fills the role of a standard file descriptor. This value must be present in the namefilter for write access to work (see private file UCAN write semantics). This fulfills the same role as an i-number in a UNIX file system.

The private partition is a pointer machine with versioned files. The i-number stays consistent between versions, and the spiral ratchet provides backwards-secret history. These are combined into a namefilter, which is used as a storage pointer (see the Namefilter section)
