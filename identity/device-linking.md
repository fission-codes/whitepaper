# Device Linking

Devices are linked by sending a symmetric read key, and delegating a UCAN to the new device's DID. There are several ways to send this information securely, but here we will be solving for the most difficult and universal case: over pubsub.

These messages are visible to the world in cleartext. We want to prevent man-in-the-middle attacks and other forms of spoofing. While we have a list of known-good exchange keys in DNS \(and later right in the user's WNFS\), we would like to avoid hittng the network as much as possible. Luckily, we can bootstrap up a secure channel with a known ID on one side, and a challenge nonce on the other.

It should be noted that the bootstrap process here may also be used to set up secure channels for other use cases, including chat.

## Sequence

### Summary

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

### **Step 1: Everyone Subscribes to Channel**

All parties listen for messags on a channel named for the root DID. A peer that can issue UCANs must be online and listening on the correct channel. For simplicity and future compatibility, it's the root DID in the UCAN chain being requested.

#### Example

💻 \(who has a UCAN already\) and 📱 \(requestor\) listen for incoming messages on channel `did:key:zALICE`

### **Step 2: Requestor Broadcasts an Exchange Public Key**

This gives everyone on the channel a 2048-bit RSA public key to send private data to.

Note that this MAY be a throwaway public key as we \(and the WeCrypto API\) keep encryption keys separate from signing keys. It will be used to bootstrap up a channel, but does not need to live beyond that \(though it can\).

#### Example

📱 broadcasts the cleartext message `did:key:zTHROWAWAY` on the channel `did:key:zALICE`

### **3. Session Key Negotiation over UCAN**

This step proves provides two things:

1. Proves that that you are talking to a machine that does in fact have the correct rights that you're looking to have delegated
2. Securely exchanges a 256-bit AES key \(which is much more efficient than RSA encryption\) for use in the rest of the session.

💻 responds by broadcasting a "closed" UCAN on channel `did:key:zALICE`, encrypted for `did:key:zTHROWAWAY`. The embedded UCAN is proof that the sender does in fact have permissions for the account, but does not delegate anything yet. The facts section \(`fct`\) includes an AES256 session key that will be used for the remainder of the communications.

```javascript
const payload = rsaEncrypt({
  from: LAPTOP_SK,
  to: IPHONE_PK, 
  payload: rsaSign({
    key: LAPTOP_SK
    payload: closedUcan
  })
})

closedUcan.iss = `did:key:z${LAPTOP}`
closedUcan.aud = `did:key:z${THROWAWAY}`
closedUcan.fct = [..., {"sessionKey": randAes256}]
closedUcan.att = [] // i.e. MUST delegate nothing
```

Here we're _securely_ responding with a randomly generated AES256 key, embedded in the UCAN's "facts" section. Since UCANs are signed, and the audience is the recipient, we have proof that this message was intended for the recipient and has not been modified along the way.

The recipient MUST validate the signature chain all the way back to the root. The first-level proofs \(non-nested\) MUST contain the permissions that you are looking to be granted.

{% hint style="danger" %}
If any of the above does not match, you MUST ignore that message. It's Eve's machine trying to establish a person-in-the-middle attack \(PITM\) 😈
{% endhint %}

### **4. Confirm Requestor PIN**

Steps 1-3 establish a connection with _a requesting machine_, but nt nessesarily _the user's machine_. To validate that this is the correct user, we go out of band, and have the human verify a code.

The requestor displays a challenge \(PIN code\) to the user. It sends the PIN and DID/signing key \(encrpyted with the AES key\) over pubsub. The UCAN holder decrypts and displays this PIN to the user and asks them to confirm that it matches. If it matches, you are talking to the correct machine, and have the DID to delegate to 🎉

#### Example

📱 receives the above message, and extracts the sender's DID \(and thus PK\). It then [verifies](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/verify) that the sender's PK is in the list of exchange keys found in DNS for the target username.

{% hint style="danger" %}
If it fails PIN validation, you MUST ignore the message, since it's Eve trying to get you to delegate rights to her 🦹‍♀️
{% endhint %}

### **5. Credential Delegation**

Now that we know that the message can be trusted, the token holder creates a UCAN with delegate rights for the requestor using their DID from the most recent message. Send that UCAN and the WNFS read key \(which is also an AES key\) back over the pubsub channel.

```javascript
aesEncrypt({
  "readKey": wnfsReadKey,
  "ucan": newUcan
})
```

