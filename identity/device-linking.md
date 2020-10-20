# Device Linking

Devices are linked by sending a symmetric read key, and delegating a UCAN to the new device's DID. There are several ways to send this information securely, but here we will be solving for the most difficult and universal case: over pubsub.

These messages are visible to the world in cleartext. We want to prevent man-in-the-middle attacks and other forms of spoofing. While we have a list of known-good exchange keys in DNS \(and later right in the user's WNFS\), we would like to avoid hittng the network as much as possible. Luckily, we can bootstrap up a secure channel with a known ID on one side, and a challenge nonce on the other.

It should be noted that the bootstrap process here may also be used to set up secure channels for other use cases, including chat.

## Summary

1. Everyone subscribes to channel
2. Requestor broadcasts public key
3. Session Key negotiation over UCAN
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

## **Sequence**

### **Step 1: Everyone Subscribes to Channel**

All parties listen for messags on a channel named for the root DID. A peer that can issue UCANs must be online and listening on the correct channel. For simplicity and future compatibility, it's the root DID in the UCAN chain being requested.

#### Example

💻 \(who has a UCAN already\) and 📱 \(requestor\) listen for incoming messages on channel `did:key:zALICE`

### **Step 2: Requestor Broadcasts Public Key**

This gives everyone on the channel a public key to send private data to.

#### Example

📱 broadcasts the cleartext message `did:key:zPHONE` on the channel `did:key:zALICE`

### **3. Session Key Negotiation over UCAN**

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

