# Certificate introduction

During the SSL/TLS handshake between a client and a server, the server will share its public key with the client.  Based on that public key, the client can setup a secure https connection to the server.

## Why do we need certificates

The client needs to make sure that the public key is being send by the real server (in our case the server where Node-RED is running)?  Because a hacker could intercept the client's https request to the server, and return his own fake public key of his own serve.  In that case the client would setup - without knowing - a secure https connection to the hacker's server.  And then the hacker can decrypt and read all the condifidential data send by the client (via its private key):

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/23459b16-e9f8-4f46-979e-4932beb5240e)

The hacker will forward all the https requests to the real server.  That way the client thinks he is directly connected to Node-RED, however there is a ***man in the middle*** intercepting all the data.

## Create a certificate

This can only be solved to add some information to our server's public key, which can be used by the client to determine that the public key belongs to the real server.  To do that, the public key needs to be signed by a trusted party (instead of using a self-signed certificate).   Such a ***certificate signing*** process goes like this:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/1321b2ba-70f9-4ec9-8bd1-b8e736544279)

1. On your server you can create a ***CSR*** (Certificate Signing Request), which contains both your public key and some metadata about your server: among other the ‘common name’ (CN) which is the hostname of the server. 
2. That CSR is send to a ***CA*** (Certificate Authority), which is a trusted organization that is certified to create digital certificates.  They create a certificate, based on a CSR to which they add the following signing information: 
   + A validity period: for most CA’s (GlobalSign, VeriSign, …) the period is 1 year.  But for LetsEncrypt it is 3 months.
   + Digital signature: they create a hash of the certificate and encrypt that hash using their private key.  Afterwards any client can also calculate that hash, which should be the same as the hash from the CA.
3. The CA returns the certificate to the server.  That certificate contains of a couple of things:
   + The public key.
   + Metadata about the server (a.o. the ‘common name’ which is the hostname of the server).
   + Signing information about the CA

## Requestor of CSR should own the domain

A hacker could create a CSR for a hostname from somebody else.  If a CA would sign such a request, then the hacker would own a certificate for a server that is owned by somebody else.  However the CA will always check whether the requestor of a certificate is also the real owner of that domain:
+ Most CA’s (GlobalSign, Verisign, …) use a manual process to check this.  They will require that the CSR is being mailed from a special administrator related email adres in the same domain as the server (e.g. admin@same_domain.com), because then they are pretty sure that the certificate requestor owns that domain.  Because a hacker can not easily create an admin email account on a company's mail server.
+ LetsEncrypt on the other hand offers free certificates via a fully automated proces:
   1. Your ***acme client*** software requests automatically (at regular time intervals) a new certificate from LetsEncrypt.
   2. LetsEncrypt returns a small challenge file to the acme client.
   3. The acme client need to make that file available on your domain (via port 80 or via a DNS record) within N minutes.
   4. When LetsEncrypt can find that file within the specified time (on the website of that domain), they know you are owner of that domain (because only a root user can install a webserver to listen to port 80 or create DNS records).
   5. Then LetsEncrypt returns the certificate to the acme client, who can start using it to establish new https connections initiated by clients.

## Self-signed certificates

A hacker could of course sign his own certificate (pretending to be some kind of CA) containing our hostname (as common name).  However when he sends a CSR to a trusted CA, they will check whether the requestor of the certificate owns the domain/hostname (which is specified int the request): see nex  Since the hacker has no root access to our domain, he cannot prove that he owns the domain.  Since the CA will reject the CSR, the hacker can only sign his certificate by himself.  Butt then he ends up with a ***self-signed certificate***, which are not considered secure by most clients (e.g. browsers, NodeJs, ...).  In other words decent https client software will reject such self signed certificates, because such clients will execute ***3 checks*** when a certificate arrives (during setup of a https connection):
+ Is the validity period of the certificate not passed yet.
+ Does the common name in the certificate match with the hostname in the url, i.e. the hostname to which we have send this request.
+ Has the certificate being signed by one of a list of trusted CA's (GlobalSign, Verisign, LetsEncrypt, ...).

For example browsers will inform you that the certificate is not valid, in this case because the common name in the certificate does not match with the hostname you have entered in the address bar (i.e. inside the url):

![image](https://github.com/user-attachments/assets/a15a8016-ebb7-4715-a3b8-e71b304fd489)
