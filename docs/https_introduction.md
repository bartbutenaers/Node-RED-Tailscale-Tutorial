# HTTPS introduction

We need to make sure that we send our data across the internet via a secure connection.  Because hackers can intercept our connection, and read the data that you are sending. This means we need a secure http connection, which means a http***s*** connection.  The https protocol is http over TLS (which is the successor of SSL), which encrypts our http packets.

Here a summary of how such a https connection is established (during a SSL/TLS handshake between the client and the server):

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/d5f2c020-0816-4037-a73a-5e3b6ed41681)
 
1. We generate once a ***keypair*** on the server, consisting of a ***private key*** and ***public key***.  These both keys are asymmetric, meaning that the public key can only be used to encrypt data and the private key can only be used to decrypt data.  Since every client is allowed to encrypt data, every client (even a hacker) can get a copy of the public key.  But the private key needs to be stored very secure on the server, because only your server is allowed to decrypt and read the data.  If a hacker could get hold of your private key, he would abuse it to decrypt your confidential information (passwords, ...) send across the network…  
2. The client starts a handshake by sending a ‘Hello’ request to the webserver.
3. The server returns the public key to the client.
4. The client generates a ***session key***, that is only used for this session.  It is a symmetric key, which means that same key can be used both for (fast) encryption and decryption.
5. The client will encrypt its session key with the server's public key, and send the encrypted session key to the server.  This is called ***key exchange***.
6. The server can decrypt the session key using its private key, and store the client's session key temporarily (as long as the session with the client exists).
7. The client will encrypt data using the session key, and send the encrypted data to the server.
8. The server will decrypt the data using the same session key.

This way the session key can be exchanged securely between the client and the server, and both can use it to encrypt and decrypt the data that they share.  As long as a hacker does not have the server’s private key, he cannot decrypt the data that he intercepts on the network...
