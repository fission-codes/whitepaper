# Versioning

FLOOFS is a nondestructive, versioned file system. This is a lot like how git history works: files that are unchanged remain unchanged, but parents need to change to accomodate any altered children.

We are using a temporal DAG with fuctional persistence. Unlike Git, it is curretly designed to be ”single writer” — new patches must be applied in order, and conflicts are not automatically resolvable. It’s like only having only one branch. That is on the roadmap, but there’s a lot of bootstrap in the meantime.

This is versioning at the file system level only. Subfile versioning can be implemented on top of FLOOFS, but does not come baked-in. This is already an improvement over what most uses expect, and each committed version of the file can be stepped through to retrieve old history \(e.g. what was the state 3 weeks ago?\).

Versioning is quite simple at present: properties such as confluence are being left in favour of easy comprehension for less technical users. A single linear history is wel understood to most people. Advanced merging and split histories are possible down the road. Collaboration is also being pushed to another layer that looks more like system memory as opposed to the FLOOFS ”disk.”

## Diffs and Generations

Let’s consider an illustrative example: there is a simple tree of depth 2.

```text
Gen 0

  A
 / \
B   C
   / \
  D   E
```

This DAG has no diffs, and one generation. Since the content changes, so does the CID and thus the pointers to it i the graph. So if we update node `E` to `E’`, the pointers above it need to be updated as well. This then changes `C` to `C’`, and thus `A` to `A’`. All other links can be copied over \(`B` and `D`\).

```text
Gen 1

  A’ <- Update link for C’ (and thus E’)
 / \
B   C’ <- Update link for E’
   / \
  D   E’ <- Change file
```

This is a new generation, which we can name after the sequence \(”generation 1”\), or by the name of the root node \(”A-prime”\). In practice, referencing the root node would be done by CID.

There is one diff here: `<A’, C’, E’>`. The rest is shared structurally. Thus, `B` originated in Generation 0, but is a member of both Generations 0 and 1.

Given that FLOOFS is nondestructive, we can easily gain history by simply adding an edge named `previous` to the older version of the vnode. This also makes validation that the DAG has been nondestructively appended with a Merkle proof that `A` exists inside `A’`.

```text
==========
= LEGEND =
==========

..... Identity relation

<———— Previous version

===========
= DIAGRAM =
===========

Gen 0          Gen 1

  A<—————————————A’
 / \            / \
B...\..........B   \
     C<—————————————C’
    / \            / \
   D...\..........D   \
        E<—————————————E’
```

At the protocol layer, this is viewed as an ever increasing, recursively nested set of DAGs with lots of shaerd structure. At the application layer, each generation is seen as a single ”slice”, and the use can pan backwards through time to previous slices — of the entire structure, some subgraph, or a single file.

{% hint style="danger" %}
This matter is somewhat complicated by the additional abstraction of an Encrypted Node, specifically the information hiding therein. Once in plaintext, it functions the same way, but there are some speical details.

Please refer to the private section docs for more.
{% endhint %}

## Rewind ⏪

It is also possible to link further back more quickly with skip lists \(though more time consuming to verify\). This would likely look something like links back in time in powers of 2 \(which is weighted towards more recent versions\). The first 10 would be:

* 1 back \(previous version\)
* 2 back
* 4 back
* 8 back
* 16 back
* 32 back
* 64 back
* 128 back
* 256 back
* 512 back

So for instance, 502 back would be achieved in 7 operations, 728 in 5. \(Calculation is: express the number in binary, and count the 1 bits\)

