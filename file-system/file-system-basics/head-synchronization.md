# Head Synchronization

The Web Native File System is a functionally persistent data structure. It's an append-only structure \(with some very minor caveats for 1. data owner _can_ overwrite to protect user sovereignty, and 2. MMPT layout is mutable at the lowest level for raw performance\).

In order to share the latest version of your data with others, the root CID needs to be broadcast. WNFS works offline, but even in an online setting is fundamentally a distributed system. Knowing if your local version is ahead of the broadcast tree, or vice vera, if very important to guard against data loss.

{% hint style="info" %}
There is nothing special about the "broadcast" part of this setup. For all intents and purposes, this can \(and in future — _will\)_ be done with `n` machines \(since versions form a partial order / semilattice\).

As such, we will name the current pair being compared as "local" and "remote".
{% endhint %}

In a fully mutable setting, this can become tricky since data is dropped. Comparing history fully depends on a history cache, loss of which leaves you needing to compare entire trees and diverge more easily. Persistent data structures have several nice properties that make approaching the problem more tractable.





