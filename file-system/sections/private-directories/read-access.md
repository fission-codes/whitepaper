# Read Access

WNFS has a recursive read access model known as a cryptree \(technically a cryptgraph in our case\). Each Decrypted Virtual Node contains the keys to its children nodes. It also includes the human-readable path name, and the of the revision that it’s aware of \(more below\).

## Deterministic Seek Ahead

Since name filters are deterministic, we can look up a version in constant time from the node cache. Below we go into greater detail about how progress is ensured, but is relevant to lookup.

If you have a pointer to a particular file, there is no way of knowing that you have been linked to the latest version of a node. The information that you do have includes everything that you need to construct a name filter.

* The current node’s identity filter
* The ratchet of the current node \(stored in the node\)
* This parent's bare name filter \(stored in the node\)

The user must always ”look ahead” to see if there have been updates to the file since they last looked. The three most common scenarios are that:

1. No changes have been made
2. There have been been a small number of changes
3. There have substantial changes since

To balance these scenarios, we progressively check for files at revision `r + 2^n` , where `r` is the current revision, and `n` is the search index. First we check the next revision. If it does not exist, we know that we have the latest version. If it does exist, check `r + 2`, then `r+4`, `r+8` and so on. Once there’s a missing version, perform a binary search. For example, if looking at a node at revision 42 that has been updated 123 times since your last recorded pointer, it takes 14 checks \(roughly `O(2 * log n)`\) to find the latest revision.

![](../../../.gitbook/assets/screen-shot-2021-06-03-at-23.46.07%20%281%29.png)

| Revision Number | Exists |
| :--- | :--- |
| 42 + 1 = 43 | Yes |
| 42 + 2 = 44 | Yes |
| 42 + 4 = 46 | Yes |
| 42 + 8 = 50 | Yes |
| 42 + 16 = 58 | Yes |
| 42 + 32 = 74 | Yes |
| 42 + 64 = 106 | Yes |
| 42 + 128 = 170 | No — First overshot! We now have an upper bound |
| 42 + 96 = 138 | Yes |
| 42 + 112 = 154 | Yes |
| 42 + 120 = 162 | Yes |
| 42 + 124 = 166 | No |
| 42 + 122 = 164 | Yes |
| 42 + 123 = 165 | Yes — No more search space, so found! |

## Lazy Progress

Anyone that can update a pointer can make permanent revision progress for themselves \(in `localStorage` or as a symlink in their FS\), or others if they have write access to this file system.

As the user traverses the private section \(down the Y-axis, across the X-axis\), they attempt to make progress in time \(forwards in the Z-axis\). If they find a node that’s ahead of a link, it updates that one link in memory. At the end of their search in 3-dimensions \(with potentially multiple updates\), they write the new paths to WNFS.

Note that they only need to do this with the paths that they actually follow! Progress in revision history does not need to be in lock step, and will converge over time.

Not all users with write access have the ability to write to the entire DAG. Writing to a subgraph is actually completely fine. Each traversal down a path will reach the most recently written node. The search space for that node is always smaller than its previous revisions, and can be further updated with other links or newer child nodes.

This contributes back collaboratively to the overall performance of the system for all users. If a malicious user writes a bad node, they can be overwritten with a newer revision by a user with equal or higher privileges. Nothing is ever lost in WNFS, so reconstructing all links in a file system from scratch is _possible_ \(though compute intensive\).

## Secret Names

Structural information can be analyzed probabilistically from a node’s name, or a tree structure. Ergo we go to some effort to hide this information from users that should not know this information.

Fully zero knowledge methods do exist for this, but are quite new and do not \(yet\) perform fast enough to be practical for this use case. WNFS opts to hide as much information as possible while remaining feasible on a low-end smartphone.

