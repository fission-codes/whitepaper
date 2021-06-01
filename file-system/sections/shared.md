# Share

There are three cases where we need to exchange data in an offline manner. Fundamentally these are all variants on ”I need to make this update, but don’t have the other party online.”

1. Shared By Me
2. Shared With Me

## Shared By Me \(authz\_out\)

Sharing information with a user that’s offline is easy thanks to authenticated key exchange. All Fission users widely distribute a list of public 2048-bit RSA keys \(their ”exchange keys”\). This is generally exposed in DNS, but also written to their file system. With these keys, exchanging read keys, UCAN credentials, and other information is possible.

The `authz_out` section of FLOOFS contains information being shared by the root user of that file system with other users. The most common case is a read key.

### Layout

This is a one-to-many exchange. Because of how account linking works, any given user will typically have a small number of exchange keys \(2-5\). Each user only has access to a single key at any given time, so the sender will use a single key to share with multiple recipient keys.

In the `authz_out` directory, there exists a flat space of directories. To make these addressable but not easily precomputable, these are named as as follows: 

```javascript
base58(SHA(sender_did ++ reciever_exchange_pk))
```

This means that there will be multiple directories for every account — one per share key.

```text
${username}.fission.name
  |
  +——authz_out
       |
       +——CPp5So2wUiJWn8hihZfy9HTp2dgq7HqJs8qauRCDpNFp
       |    |
       |    +——Cn415bWgLmnCyVdY7S1EkBUhxmhbg5ogJCqbAzHoeZx6.json.dh
       |    |
       |    +——9HpgagjGnjdrBoEj4aCFJaZZCLQP78Fqk2JmzxHAS65B.json.dh
       |
       +——CxJWPa1ZnSFoGEYwKKQYGgTJ85LPR1oubHNExVgSzgib
            |
            +——AkcF9bfxpK3zGhNNM8wdr7N9EApqSZ6xqRnBDHbkVSsv.json.dh          
```

Inside the directory are one or more encrypted files with the name of the sender’s public key which was used to encrypt it. When a recipient decrypts this file, they are expected to copy the information to their `authz_out` directory in their own FLOOFS, or at least cache the  information on their local system.

### Content

The content of these files is a very straightforward JSON array containing UCANs or read keys. UCANs are described elsewhere. Read keys look like this:

```typescript
interface SharedKey {
  cipher:  KeyType; // AES256
  key:     Bytes;
  pointer: NameFilter;
}
```

## Shared With Me \(authz\_in\)

The inverse of `authz_out` is `authz_in`, where the FLOOFS root user is the recipient. This is their cache of keys, credentials, and encrypted pointers that have been shared with them.

This is important to have in case a local cache is cleared, they link another machine, or the original copy in a remote `authz_out` is removed. The mechanism is straightforward: information is kept in a single file, encrypted with an AES256 key.

This AES key is distributed to all linked instances. This key can be rotated at any time \(e.g. if comprimised\). As such, a copy is also shared with all ofthe user’s exchange keys. The sender is signaled by the name of the directory \(a hashed value for space reasons\).

```text
${username}.fission.name
  |
  +——authz_in
       |
       +——shared.json.crypt
       |
       +——G9N5KiVYLKbQLpTsnaTgr5E8kt95F56G7oLMhPryVz1m
            |
            +——4GVjkeLXt1h3xnay5dVF6Ekdjw9RqYFKYH3rUjk9CHHy.aes256.dh
            |
            +——7ZXatFQWw3tJsDHhozotH6V4D8vKqjenTGf2sz3XfFzA.aes256.dh
            |
            +——FkBvWMAy14HpwBtN9wFMmEYDErgzp5KiBragAQPsib1Z.aes256.dh
```

