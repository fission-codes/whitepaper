# Device Linking

## Summary

1. Everyone subscribes to channel
2. Requestor broadcasts public key
3. Key negotiation over UCAN
4. Confirm requestor PIN
5. Credential delegation

### Dramatis Personae

#### People

* Alice 👩‍💻
  * The root user
* Eve 🦹‍♀️
  * An attacker trying to gain access to Alice’s account

#### Machines

* Desktop 🖥
  * Alice’s original “root” device
  * Does not participate in the example flow, but exists offline
  * Has a PK corresponding to `did:key:zALICE`
* Laptop 💻
  * Alice’s second device
  * Has a PK corresponding to `did:key:zLAPTOP`
  * Has an existing UCAN for Alice’s account
* Phone 📱
  * Alice’s iPhone
  * The device requesting linking for Alice’s account
  * Has a PK corresponding to `did:key:zPHONE`
* Evil Server 😈
  * Eve’s server that watches all traffic on pubsub

## **Steps**

All of these messages are being conducted over IPFS pubsub, and are visible to the world in cleartext. We want to prevent man-in-the-middle attacks and other forms of spoofing. Luckily, we have a list of known-good exchange keys in DNS \(and later right in the user's WNFS\).

The downside here is that we need to wait for DNS to update with the exchange keys. We already need to do this with the root DID, so in theory not much changes time-wise.

### **1. Everyone Subscribes to Channel**

All parties listen for messags on a channel named for the root DID.

#### Scenario

:computer: \(who has a UCAN already\) and :iphone: \(requestor\) listen for incoming messages on `did:key:zALICE`

A peer that can issue UCANs must be online and listening on the correct channel. For simplicity and future compatibility, it's the root DID in the UCAN chain being requested.

### **2. Requestor Broadcasts PK**

djskal

#### Scenario

📱 broadcasts the cleartext message `did:key:zPHONE` on the channel `did:key:zALICE`

This gives all recipients the public key for :iphone: to send encrypted messages back to

### **3. Key Negotiation over UCAN**

:computer: responds by broadcasting the following message on channel `did:key:zALICE`

```javascript
{
  "sender": "did:key:zLAPTOP",
  "sessionKey": <rsa_enc(LAPTOP_SK,IPHONE_PK,rand_aes256)>
}
```

Here we're _securely_ responding with a randomly generated AES256 key. The payload is a single string, but it has encoded the information on

Let's break that payload down:

* It's encoded in JSON. Less compact, but also easier for humans to implement & debug.
  * PLEASE LET ME KNOW if it's easier in a stringified form with escaped quotes.
* `did:key:zLAPTOP` is simply the _cleartext_ DID of the sender \(i.e. its PK\)
* `rsa_enc(LAPTOP_SK,IPHONE_PK,rand_aes256)` is an RSA key exchange from the laptop's SK to the phone's PK. It's payload is a _randomly generated_ AES256 \(symmetric\) key that we will use for the rest of the session.
* Make sure that the sender stores the AES key. You'll need it soon :key: 

### **4. Confirm Requestor PIN**

📱 receives the above message, and extracts the sender's DID \(and thus PK\). It then [verifies](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/verify) that the sender's PK is in the list of exchange keys found in DNS for the target username.

If it fails validation, ignore the message, since it's Eve trying to mess with your security :woman\_supervillain:

### **5. Credential Delegation**

Now that we know that the message can be trusted, :iphone: decrypts it with their SK to recover the AES key from earlier.

Store this key locally, since it will be used for all future communication in this session.

