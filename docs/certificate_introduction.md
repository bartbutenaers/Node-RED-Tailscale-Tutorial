# Certificate introduction

During the SSL/TLS handshake between a client and a server, the server will return its public key.  Based on that public key, we can setup a secure https connection between the client and the server.

But how do we know that the public key that we receive is being send from the real server (in our case the server where Node-RED is running)?  Because a hacker could intercept our https request to the server and send his own fake public key, pretending it to be our server's public key.  In that case we would setup a secure https connection, but not to the real server.  Instead we would setup a secure https connection to the hacker’s server, where all our condifidential data could be decrypted by the hacker (via its private key):

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/23459b16-e9f8-4f46-979e-4932beb5240e)

This can only be solved to add some information to our server's public key, so we know that the received public key belongs to our own server.  To do that, our public key needs to be signed by a trusted party.   Such a ***certificate signing*** process goes like this:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/1321b2ba-70f9-4ec9-8bd1-b8e736544279)

1. On your server you can create a ***CSR*** (Certificate Signing Request) containing both your public key and some metadata about your server: for example the ‘common name’ (CN) which is the hostname of the server. 
2. That CSR is send to a ***CA*** (Certificate Authority), which is a trusted organization that is certified to create digital certificates.  They create a certificate, based on a CSR to which they add the following signing information: 
   + A validity period: for most CA’s (GlobalSign, VeriSign, …) the period is 1 year.  But for LetsEncrypt it is 3 months.
   + Digital signature: they create a hash of the certificate and encrypt that hash using their private key.  Afterwards any client can also calculate that hash, which should be the same as the hash from the CA.
3. The CA returns the certificate, which is a combination of a couple of things:
   + The public key.
   + Metadata about the server (a.o. the ‘common name’ which is the hostname of the server).
   + Signing information about the CA

A hacker could of course sign his own certificate (pretending to be some kind of CA), but then he ends up with a self signed certificate.  Such certificates are not considered secure by most clients (e.g. browsers).  Because decent https client softwares will execute ***3 checks*** when a certificate arrives when a https connection is being initiated:
+ Is the validity period of the certificate not passed yet.
+ Does the common name in the certificate match with the hostname in the url, i.e. the hostname to which we have send this request.
+ Has the certificate being signed by one of a list of trusted CA's (GlobalSign, Verisign, LetsEncrypt, ...).

A hacker could of course also create CSR for a hostname from somebody else.  If a CA would sign it, then he would own a certificate for a server that is owned by somebody else.  However the CA will always check whether the requestor of a certificate is also the real owner of that domain:
+ Most CA’s (GlobalSign, Verisign, …) use a manual process.  They will require that the CSR is being mailed from a special administrator related email adres in the same domain as the server (e.g. admin@same_domain.com), because then they are pretty sure that the certificate requestor owns that domain.
+ LetsEncrypt offers free certificates via a fully automated proces:
   1. Your acme client software requests automatically (at regular time intervals) a new certificate from LetsEncrypt.
   2. LetsEncrypt returns a small challenge file to the acme client.
   3. The acme client need to make that file available on your domain (via port 80 or via a DNS record) within N minutes.
   4. When LetsEncrypt can find that file in time (on the website of that domain), they know you are owner of that domain (because only a root user can listen to port 80 or create DNS records).
   5. Then LetsEncrypt returns the certificate to the acme client, who can start using it to establish new https connections initiated by clients.
