# Attenuation

Attenuations are a thin layer on top of the UCAN spec presented earlier in this paper. Each type of resource has its own mutation semantics that can be expressed in a UCAN. For example, control of an email address confers the ability to send email, but not to deploy a website to it — that doesn't make sense in the context of email.

The pages that follow include attenuation semantics for our first-party resources.

It is worth mentioning that there is an implicit set of "null" resources. These are anything that the root DID has created or "owns" in some other way. They attenutate these without proof \(the introduction rule\).

