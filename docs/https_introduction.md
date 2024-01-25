# HTTPS introduction

We need to have a secure http connection, which means a https connection.  In that case all the data will be send encrypted across the network, so that a hacker cannot intercept and read the data. 

Summarized how such a https connection is established:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/d5f2c020-0816-4037-a73a-5e3b6ed41681)
 
1. We generate once a keypair on the server, consisting of a private key and public key.  These keys are asymmetric, meaning that the public key can be used only to encrypt data and the private key can only be used to decrypt data.  Since every client is allowed to decrypt data, every client (even a hacker) may have the public key.  But the private key needs to be stored very secure on the server, because a hacker could abuse it to decrypt confidential information send across the network…
2. The client starts a handshake by sending a ‘Hello’ request to the webserver.
3. The server sends the public key to the client.
4. The client generates a session key, that is only used for this session.  It is a symmetric key, which means it can be used both for (fast) encryption and decryption.
5. The client will encrypt its session key with the public key, and send the encrypted session key to the server.  This is called key exchange.
6. The server can decrypt the session key using its private key, and store the session key.
7. The client will encrypt data using the session key, and send the encrypted data to the server.
8. The server will decrypt the data using the same session key.

As long as a hacker does not have the server’s private key, he cannot decrypt the data that he intercepts on the network...
