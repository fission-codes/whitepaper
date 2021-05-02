# Challenge-Based

As a method of providing compatibility layer with legacy systems, challenge-based authentication schemes are possible via signing a vouching claim with an HMAC. This identity is as trustworthy as the third-party that is managing the identity for them.

A practical use case example is interoperating with a brownfield service which relies on [OIDC](https://en.wikipedia.org/wiki/OpenID#OpenID_Connect). The entire chain may be self-certifying, except for the portions that are rooted in an OIDC identity. This is secured with an HMAC and a check with the trusted auth server.

