# Versioning

WNFS is a nondestructive, versioned file system. This is similar to Git history: files that are unchanged \(shared bwteen versions\) remain unchanged, but parents include links to them, their previous version, and to new or altered altered children.

WNFS uses a temporal graph with functional persistence. It is designed to be ”single writer” — new patches must be applied in order, and conflicts are automatically resolvable with a rebase / last-writer-wins \(LWW\) strategy, or with software transaction memory \(STM, see relevant section\).

This is versioning at the file system level only. Subfile versioning can be implemented on top of WNFS with STM, or separately with techniques such as CRDTs \(coming baked in to the database layer\). This is an improvement over what most uses expect, and each committed version of the file can be stepped through to retrieve old history \(e.g. retrieve the state of a file from many weeks ago\).

Temporality / versioning is intentionally quite straightforward. As such, properties such as confluence are being left in favour of easy comprehension and performance. A single linear history is well understood by most people. Advanced merging and split histories are possible, but there is no current plan to support them at the file system layer \(though it can be achieved with the Datalog-driven database\).

## Diffs, Revisions, and Generations

Let’s consider an illustrative example: there is a simple tree of depth 2.

```text
Gen 0

  A
 / \
B   C
   / \
  D   E
```

This DAG has no diffs, one revision, and one generation. Since the content changes, so does the CID and thus the pointers to it i the graph. If we update node `E` to `E’`, the pointers above it need to be updated as well. This then changes `C` to `C’`, and thus `A` to `A’`\(“A revision 1”\). All other links can be copied over \(`B` and `D` remain at their previous revision 0\). We are using the term ”revision” to avoid the word ”version” which conflicts with the version of WNFS itself.

```text
Gen 1

  A’ <- Update link for C’ (and thus E’)
 / \
B   C’ <- Update link for E’
   / \
  D   E’ <- Change file
```

As a rule, the highest node in a \(sub\)graph will have the highest version number, potentially beyond the highest version leaf node. This is because the parent is aggregating changes across multiple independent subgraphs.

This is a new generation, which we can name after the sequence \(”generation 1”\), or by the name of the root node \(”A-prime”\). In practice, referencing the root node would be done by CID.

There is one diff here: `<A’, C’, E’>`. The rest is shared structurally. Thus, `B` originated in Generation 0, but is a member of both Generations 0 and 1.

Given that WNFS is nondestructive, we can easily gain history by simply adding an edge named `previous` to the older version of the vnode. This also makes validation that the DAG has been nondestructively appended with a Merkle proof that `A` exists inside `A’`.

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

Because every version has links back, this setup gives `O(log n)` history access. So for instance, 502 back would be achieved in 7 operations, 728 in 5, and 1M in 7. \(Calculation: express the number in binary, and count the 1 bits\).

