# Private

The private section is an extension of the functionality in the public section. The additional goals of the private tree are to maximize privacy, security, flexibility, and autonomy for users.

It adds an additional aspect of secrecy to the boundary between the data and file layers. private data is kept in a flat, scrambled tree, containing encrypted contents. A user that is allowed to decrypt a node in this structure is able to progressively discover the semantic file and directory structure contained in these nodes.

![](../../../.gitbook/assets/screen-shot-2021-06-01-at-22.05.04%20%281%29.png)

{% hint style="danger" %}
Write access in the WNFS is a semi-trusted setup. The root user delegates write access to subgraph. WNFS has aimed to be correct-by-construction where possible. However, since service verifiers cannot validate the inner contents of a node beyond aspect of the file name, the submitter/writer may fill that node with anything.

Validating the contents homomorphically or with a zero knowledge setup is technically possible, but the technology is still early, slow, and expensive. At minimum the efficiency need to improve greatly before that’s viable on deep homomorphic DAG operations on a consumer device.
{% endhint %}

