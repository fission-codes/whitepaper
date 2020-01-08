# DID Document

Since our DID method is a single, public key, always on the Edwards curve, the DID document itself may be reconstructed from a public key at any time. This layer exists purely for compatibility purposes.

## Complete Example Document

DID documents of this type always follow this exact format:

```javascript
{
  "@context": "https://w3id.org/did/v1",
  "id": "did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH",
  "publicKey": [{
    "id": "did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH#pubkey",
    "type": "Ed25519VerificationKey2018",
    "controller": "did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH",
    "publicKeyBase58": "B12NYF8RrR3h41TDCTJojY59usg3mbtbjnFs7Eud1Y6u"
  }],
  "authentication": [ 
    "did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH#pubkey"
  ],
  "assertionMethod": [ 
    "did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH#pubkey"
  ],
  "capabilityDelegation": [
    "did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH#pubkey"
  ],
  "capabilityInvocation": [
    "did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH#pubkey"
  ],
  "keyAgreement": [{
    "id": "did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH#kakey",
    "type": "X25519KeyAgreementKey2019",
    "controller": "did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH",
    "publicKeyBase58": "JhNWeSVLMYccCk7iopQW4guaSJTojqpMEELgSLhKwRr"
  }]
}
```

[Example Source](https://digitalbazaar.github.io/did-method-key/)

## `@context`

The `@context` key simply states which version of the spec this DID conforms to.

For example: `'https://w3id.org/did/v1'`

## `id`

The `id` is the concatenation of the method `did:key:` with the public key.

For example

```javascript
'did:nacl:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI'
```

## `publicKey`

The `publicKey` array MUST:

* Contain EXACTLY one key record
* The public keys MUST be identical
* MUST use the index `#pubkey`
* MUST have the type `ED25519SignatureVerification2018`
* Be encoded in base64 \(for transmission and consistency\)

Example

```javascript
"publicKey": [{
  "id": "did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH#pubkey",
  "type": "Ed25519VerificationKey2018",
  "controller": "did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH",
  "publicKeyBase58": "B12NYF8RrR3h41TDCTJojY59usg3mbtbjnFs7Eud1Y6u"
}]
```

{% hint style="warning" %}
This contains a lot of redundant information, but is required to be compatible with other DID services
{% endhint %}

Unlike other DID methods, a Fission Identity MUST contain _exactly one_ key. Fission treats identity as pseudonymous at best, and thus relegates access control to the realm of verifiable claims.

> There is a need for non updateable DID's for use in IOT and other applications, where lack of network, size of code base and other such concerns are paramount to adoption. These concerns need to be addressed while not lowering the overall security guarantees.
>
> ~The [uPort NaCL README](https://github.com/uport-project/nacl-did)

Our motivation is focused primarily on universality. Rather than updating a document over time, the DID is minimal, immutable, and may be reconstructed from a key pair at any time.

## `authentication`

The `authentication` array MUST:

* Contain EXACTLY one key record
* The key MUST be identical to `#pubkey`

```javascript
"authentication": [ 
  "did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH#pubkey"
]
```

## 

