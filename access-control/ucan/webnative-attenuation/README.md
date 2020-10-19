# Attenuation

Attenuations for the WebNative File System are a thin layer on top of the UCAN spec presented earlier in this paper.

Each type of resource has its own mutation semantics that can be expressed in a UCAN. For example, control of an email address confers the ability to send email, but not to deploy a website to it — that doesn't make sense in the context of email.

The pages that follow include attenuation semantics for our first-party resources.

