# Keys & Pointers

Query access is mediated entirely by references and cryptographic keys. Having both of these is the definition of ”having access rights.”

### Elements

1. Reference \(e.g. content address\)
2. Decryption key \(e.g. AES-256 symmetric key\)

## A Note on WNFS Interaction

The Web Native File System \(AKA ”WNFS” — found in its own section of this whitepaper\) is built on top of this style of read access, by using a technique known as “[cryptrees](https://raw.githubusercontent.com/ianopolous/Peergos/master/papers/wuala-cryptree.pdf)“.

In the private section, each directory contains pointers and keys for all of its children. As such, granting access to a directory also grants access to all of its children. This is granted “merely” by the parent relation.

