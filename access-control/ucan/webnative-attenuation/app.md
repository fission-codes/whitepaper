# Web Application

## Resource

The resource type is `”webapp”`. The resource value is a DNSLink pointing at the highest node in the graph granting access. Everything below is given the same acces \(as its content\).

## Paths

All paths in WNFS are gven in relationship to some head pointer via a [DNSLink](https://docs.ipfs.io/concepts/dnslink/). Access is given by a binary `OR` of the path, and match that to the longer path. If they match, access is granted.

Trailing slashes on directories are OPTIONAL, though recommended for clarity. The verifier MUST infer a trailing slash. This is to prevent matching 

