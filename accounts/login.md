# Login

The concept of "logging in" conveys the ability of a user to access and/or modify resources.

This has historically been done with some proof of identity, via a password or other secret. More sophisticated systems have used challenges to prove ownership of email, public key signatures, or zero knowledge proofs, and biometrics to improve security and convenience. To wit, the metaphor is one of identity \("I log in to _my_ account _provided by_ an app"\) rather than one of action \("I am _empowered_ to complete _my task_"\).

As mentioned elsewhere, Fission has an intentionally weak concept of "identity". This is a system more on "authorization" than "authentication". A user may be said to have "logged in" to an app if they have the necessary permissions available to perform actions as that user for their current application.

However, the app is an agent of the user, and its codes ultimately performs actions on the users behalf. Perhaps a better view is that the user empowers the app with the delegated capabilities required to carry out tasks on their behalf. It is a model where the human is in control, not the app.

## 

