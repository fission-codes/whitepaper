# Login

## What Even Is a "Login"?

As mentioned in its own section, Fission has a weak concept of "identity"; one more based on "authorization" than "authentication". A user may be said to have "logged in" to an account if they have the necessary permissions available to perform actions as that user for their current application.

## Mechanism

 to  in to an acocunt Devices are linked by sending signing a symmetric WNFS read key and delegating a UCAN to the new device's DID. There are several ways to send this information securely, but here we will be solving for the most difficult and universal case: over pubsub.

These messages are visible to the world in cleartext. We want to prevent man-in-the-middle attacks and other forms of spoofing. While we have a list of known-good exchange keys in DNS \(and later right in the user's WNFS\), we would like to avoid hitting the network as much as possible. Luckily, we can bootstrap up a secure channel with a known ID on one side and a challenge nonce on the other.

It should be noted that the bootstrap process here may also be used to set up secure channels for other use cases, including chat.

## 
