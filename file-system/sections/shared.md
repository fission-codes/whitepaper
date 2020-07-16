# Share

There are three cases where we need to exchange data in an offline manner. Fundamentally these are all variants on ”I need to make this update, but don’t have the other party online.”

1. Shared By Me
2. Shared With Me
3. Self-shared

## Shared By Me

Sharing information with a user that’s offline is easy thanks to Diffie Helman key exchange. All Fission users widely distribute a list of public 2048-bit RSA keys \(their ”exchange keys”\). This is generally exposed in DNS, but also written to their file system. With these keys, exchanging read keys, UCAN credentials, and other information is possible.

The `shared_by_me` section of FLOOFS contains information being shared by the root user of that file system with other users. The most common case is a read key.

### Layout

This is a one-to-many exchange. Because of how account linking works, any given user will typically have a small number of exchange keys \(2-5\). Each user only has access to a single key at any given time, so the sender will use a single key to share with multiple recipient keys.

In the `shared_by_me` directory, there exists a flat space of directories. To make these addressable but not easily precomputable, these are named as as follows: 

```javascript
base58(SHA(sender_did ++ reciever_exchange_pk))
```

This means that there will be multiple directories for every account — one per share key.

```text
${username}.fission.name
  |
  +——shared_by_me
       |
       +——CPp5So2wUiJWn8hihZfy9HTp2dgq7HqJs8qauRCDpNFp
       |    |
       |    +——Cn415bWgLmnCyVdY7S1EkBUhxmhbg5ogJCqbAzHoeZx6.dh
       |    |
       |    +——9HpgagjGnjdrBoEj4aCFJaZZCLQP78Fqk2JmzxHAS65B.dh
       |
       +——CxJWPa1ZnSFoGEYwKKQYGgTJ85LPR1oubHNExVgSzgib
            |
            +——AkcF9bfxpK3zGhNNM8wdr7N9EApqSZ6xqRnBDHbkVSsv.dh
          
```

Inside the directory are one or more encrypted files with the name of the sender’s public key which was used to encrypt it. When a recipient decrypts this file, they are expected to copy the information to their `shared_with_me` directory in their own FLOOFS, or at least cache the  information on their local system.

The content of this file is one or more records described below.

### Read Key Exchange

