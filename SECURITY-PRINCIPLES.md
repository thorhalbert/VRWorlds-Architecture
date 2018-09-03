# Security Principles for VRWorlds Architecture

## Identity

* We don't care who you are in the real world.  We only care that you have a permanent identity that is anchored.  You are your Kudo Server's GUID and your own Avatar's Aspect GUID signed by yourself and your Kudo Server.  
  * There is a concept of the "Prime Aspect", which tends to correspond to your real self, but this is deeply hidden.  Its intent is primarily for dealing with real world things, like signing contracts and buying or renting real-world property.
* Kudo servers are their own thing.  They don't have to be identified beyond themselves.  They sign their own GUID (with an intermediate certificate).  They are their GUID and their public root certificate key.   Someone spoofing your guid would have a different root certificate.
* The same is true of all servers (Entity, World, and Avatar).   It is permitted however for all of these servers to have the same GUID if they're all operated the same organization.   In the current version of the browser, these would run in the same V8 server.
* Each Aspect (every Avatar can have as many Aspects as desired, with whatever level of linkage is desired) and every type of server has a Crypto Certificate (public and private).  
  
## Kudo Server Security

* Kudo servers create and maintain crypto certificates for its clients.   These are the base servers, Avatars, Entities, and Worlds.   Also, they serve as the primary identity backing for the Avatar servers for the individual aspects.
* Kudo servers do not ever give out root certificates for any signer.  They give out short-lived intermediate signing certificates (we're thinking about a 2 week lifetime, though this might eventually be much shorter if feasible--say an hour)
* The current thought (without an HSM) is to keep the primary secrets, the main intermediate signing certs from the root cert (which will never be kept online, even passphrased), are kept in some sort of encrypted service like HashiCorp's Vault.   Since we'll probably be using the open version of Mongo for storage, private keys which are stored will be passphrased by a "passphrase of the week", which will also be kept in the Vault (if the passphrase algorithm is not an adequate encryption we'll do some otherwise encryption-at-rest via AES).
* Upon sufficient authentication, the server or browser which is managing your Aspect's will be given a short-term intermediate certificate by the Kudo Server.   We're not sure how robust the windows keystore (or perhaps the linux keystore is), but that's the current idea for storage, though the linux servers can put these in Vault.   We've been thinking about a two-factor passphrase which might be tied to a phone app.   We don't really want users to directly have to deal with passwords and passphrases, though they might need one to log in the Kudo server.   Passwords are ultimately a disaster.

## Unknown Issues

* How good is the windows keystore?
* What encryption is used for cert passphrases, and is this robust?  It's DES or 3DES--it's a bit primitive.  There may be implementations with AES.   OpenSSL has AES and Camelia encryptors.
* How does Vault compare to a real HSM?
