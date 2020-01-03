# DID Document

## DID Document

Since the NaCl method is a single, public key, always on the Edwards curve, the DID document itself may be reconstructed from a public key at any time. This layer exists purely for compatibility purposes.

### Keys

#### `@context`

The `@context` key simply states which version of the spec this DID conforms to.

For example: `'https://w3id.org/did/v1'`

#### `id`

The `id` is the concatenation of the method `did:nacl:` with the public key.

For example

```javascript
'did:nacl:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI'
```

#### `publicKey`

The public key array MUST:

* Contain _exactly one_ key
* Be encoded in base64 \(for transmission and consistency\)
* Follow this format:

```javascript
{
  id: 'did:nacl:myPublicKey',
  type: 'ED25519SignatureVerification',
  owner: 'did:nacl:myPublicKey',
  publicKeyBase64: 'myPublicKey'
}
```

{% hint style="warning" %}
This contains a lot of redundant information, but is required to be compatible with other DID services
{% endhint %}

Unlike other DID methods, a Fission Identity MUST contain _exactly one_ key. Fission treats identity as pseudonymous at best, and thus relegates access control to the realm of verifiable claims.

> Motivation. There is a need for non updateable DID's for use in IOT and other applications, where lack of network, size of code base and other such concerns are paramount to adoption. These concerns need to be addressed while not lowering the overall security guarantees.
>
> ~The [uPort NaCL README](https://github.com/uport-project/nacl-did)

Our motivation is focused primarily on universality. Rather than updating a document over time, the DID is minimal, immutable, and may be reconstructed from a key pair at any time.

#### `authentication`

The `authentication` array MUST:

* Contain exactly one key
* This key must be the same as in the `publicKey`
* Use the index `#key1` 
* Specify the `ED25519SigningAuthentication` scheme

```javascript
{
  type: 'ED25519SigningAuthentication',
  publicKey: `did:nacl:myPublicKey#key1`
}
```

### Example Document

```javascript
{
  '@context': 'https://w3id.org/did/v1',
  id: 'did:nacl:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI',
  publicKey: [
    {
      id: `did:nacl:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI#key1`,
      type: 'ED25519SignatureVerification',
      owner: 'did:nacl:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI',
      publicKeyBase64: 'Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI'
    }
  ],
  authentication: [
    {
      type: 'ED25519SigningAuthentication',
      publicKey: `did:nacl:Md8JiMIwsapml_FtQ2ngnGftNP5UmVCAUuhnLyAsPxI#key1`
    }
  ]
}
```

## 

