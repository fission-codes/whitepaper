# Private

The private section is an extension of the functionality in the public section. The additional goals of the private tree are to maximize privacy, security, flexibility, and autonomy for users.

{% hint style="danger" %}
Write access in the WNFS is a semi-trusted setup. The root user delegates write access to subgraph. WNFS has aimed to be correct-by-construction where possible. However, since service verifiers cannot validate the inner contents of a write beyond access to a \(hidden\) file path, the submitter/writer may fill that node with anything.

Validating the contents homomorphically or with some zero knowledge setup is technically possible, but the technology is still early. At minimum the efficiency need to improve greatly before that’s viable on deep DAG writes on a low-powered smartphone.
{% endhint %}

