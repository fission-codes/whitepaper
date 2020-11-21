# Head Synchronization

The Web Native File System is a functionally persistent data structure. It's an append-only structure \(with some very minor caveats for 1. data owner _can_ overwrite to protect user sovereignty, and 2. MMPT layout is mutable at the lowest level for raw performance\).

In order to share the latest version of your data with others, the root CID needs to be broadcast. WNFS works offline, but even in an online setting is fundamentally a distributed system. Knowing if your local version is ahead of the broadcast tree, or vice vera, if very important to guard against data loss.

{% hint style="info" %}
There is nothing special about the "broadcast" part of this setup. For all intents and purposes, this can \(and in future — _will\)_ be done with `n` machines \(since versions form a partial order / semilattice\).

As such, we will name the current pair being compared as "local" and "remote".
{% endhint %}

In a fully mutable setting, this can become tricky since data is dropped — you diverge _immediately_.You can work around this by comparing a history log \(as we do for the private section\). Persistent data structures have several nice properties that make approaching the problem more tractable.

## WNFS Root

The root of the file system itself is designed to be very flexible, and support many different versioning metds below it, specifically:

* Structural \(`public`\)
* Log \(`private` due to for security\)
* Mutable \(`pretty` & `shared`\)

As such, you need to look at  the sections themselves to determine priority. If one section is ahead of remote, and the other is behind remote, then this is considered to have diverged, and user intervention is required. This is actually not as bad as it sounds, since the actual data content would be the same even if comparing a versioned root. It feels off because we're treating the sections differently, but they're functionally equivalent.

## Structural Versioning



