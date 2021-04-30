# Object Capability Model

> The structural properties of object capability systems favor modularity in code design and ensure reliable encapsulation in code implementation.  
>   
> These structural properties facilitate the analysis of some security properties of an object-capability program or operating system. Some of these — in particular, information flow properties — can be analyzed at the level of object references and connectivity, independent of any knowledge or analysis of the code that determines the behavior of the objects. As a consequence, these security properties can be established and maintained in the presence of new objects that contain unknown and possibly malicious code.
>
> — [Wikipedia](https://en.wikipedia.org/wiki/Object-capability_model#Advantages_of_object_capabilities)

Access control is mostly widely achieved via access control lists \(ACLs\): metadata or database tables that specify which user is allowed to perform which actions. This requires that the application boundary enclose all of the data and users. In fully centralized systems, this works reasonably well since all requests go through the same controlling source. However, as real world use cases inevitably expand, ACLs need to cover increasing complexity, exceptions, and roles. It often becomes error prone to write and maintain, and makes the ACL subsystem itself a target for attackers.

The object capability \(OCAP\) model works in the opposite mode. While ACLs are _reactive & centralized_, OCAP is _proactive & decentralized_ — an agent is allowed to perform some action if they have poof of those rights. This makes access control very granular, work offline, and in certain variants \(like ours\) empowers users to delegate rights to others to act on their behalf. This greatly simplifies access control by presenting a document that includes what the user is permitted to do. The complexity arises from managing which credentials exist, and revoking them. The standard approch is to maintain a public revocation list, which is checked on each request. We will go into that in more depth later.

Fission uses a blend of correct-by-construction read authorization and cryptographically-secured write certificates. This is a trustless method suitable to centralized, decentralized, peer-to-peer, and local-first applications. In essence, it means using a mix of signature chains to control who has access to what. Read access is granted by the mere fact that someone has the correct decryption key. Write access is mediated with signature chains, in a similar way to the common X.509 certificate. Rather than have a centralized application handle access control, this is usable everywhere, and under any circumstance.

{% hint style="info" %}
It should be noted that Fission’s authorization system relies heavily on capabilities and less on identity. The concept of ”identity” is weak. A user holding at least all of the capabilities as the resource owner can be considered equal in all respects with regard to authorization.
{% endhint %}

