---
description: >-
  Text and images on this page are adapted from
  https://github.com/qri-io/wnfs-go/blob/master/doc/cipherchunk.md#05-security-concerns.
  They were contributed by Brendan O'Brien (@b5), and modified by @ex
---

# Cipher Chunking

![Source: https://github.com/qri-io/wnfs-go/blob/master/doc/cipherchunk.md#05-security-concerns](../../../../.gitbook/assets/fig01-size\_chunking.png)

IPFS stores data in Merkle DAGs: directed acyclic graphs that refer to content by hash-of-content. Turning an arbitrary byte sequence into a Merkle DAG requires breaking up the input sequence into sub-sequences that can be hashed and arranged into a tree. The process of breaking the input sequence into sub-sequences is called _chunking_:

![Source: https://github.com/qri-io/wnfs-go/blob/master/doc/cipherchunk.md#05-security-concerns](../../../../.gitbook/assets/fig02-cipher\_chunking.png)

_Cipher Chunking (CCK)_ is a modified size-chunking strategy that passes chunked data through an encryption step before hashing into blocks.

We construct a chunker that wraps a _cipher_ built from a single _symmetric key_ (usually a random number 32 bytes in length). The key must be kept secret, and stored separately from the data. Each chunk gets a cryptographicly random _initialization vector (IV)_ when it's constructed. The IV is often referred to as a _nonce_. Nonces are not secret, but must be retained and combined with the key to decrypt the ciphertext.

By convention the nonce is prepended to the first bytes of the block ciphertext. This is a common convention used when storing block-encrypted data because the nonce is required before decryption can begin. The Go language `crypto/cipher` standard libary package prepends nonces to ciphertext by default. Ciphers use a nonce size that is based on the length of the key, making the process of separating nonce from ciphertext trivial.

CC has a few notable properties:

* The DAG structure is _not_ encrypted, only the byte sequence is.
  * This is a _small_ compromise in secrecy. Encrypted data is normally stored in an ordered sequence, where the block size is itself a secret
    * In this case an attacker knows that each block begins with a nonce.
  * All existing IPFS code will interpret cipherchunked as a valid UnixFS file, the data will be illegible without the key.
  * IPFS UnixFS files support `Seek`ing to byte offsets by skipping through the dag. cipherchunked files inherit this property.
  * Security _could_ be added by randomizing & encoding special instructions for walking the DAG. It's not clear this would provide much benefit.
* CCKing is DAG-layout independent. Balanced & Trickle DAG structures can be constructed using existing dag layout construction code.
* Hashing the same content multiple times will produce different hashes, because nonces are randomly generated and stored within blocks. This is required for keeping encrypted data secure, regardless of how encrypted data is stored.
  * Because of this **encrypted data stored on IPFS will never "self-deduplicate" through hash collision the way plaintext does.**
* CCKing does not support UnixFS Directories. Directories must be defined in plaintext.
* CCKing files _can_ be stored within existing plaintext directories
* CCKing can be used equally with both block and streaming ciphers. It aligns byte length to IPFS chunk size

**Performance**

When writing to IPFS data _must_ be chunked. If we want to write encrypted data to IPFS, it _must_ be encrypted, cipherchunking combines these two processes into the same step, decreasing the overall amount of memory allocations required when compared to encrypting data before chunking by re-using memory allocations already required by the chunker for encryption.

Combining these steps drives optimizations in either chunking or encryption into the other. A parallelized DAG constructor when combined with a cipherchunker will yield parallelized encryption, **even if the chosen cipher doesn't support parallelized encryption**.
