# Certificate introduction

But if our client sends a request to the server, how do we know that the public key that we receive is from the server and not from a hacker?  Because a hacker could intercept our request and send his own public key.  Then we have a secure https connection, but to the hacker’s server where he can decrypt and read all our condifidential data:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/6037d033-29fb-452e-ae97-882d143e588e)

This can only be solved to add some information to our public key, so we know that the public key belongs to our server.  That process goes like this:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/1321b2ba-70f9-4ec9-8bd1-b8e736544279)

1. On your server you can create a CSR from your public key and some metadata, for example the ‘common name’ (CN) which is the hostname of the server. 
2. That CSR is send to a CA, which is a trusted organization that is certified to create digital certificates.  They do that based on a CSR, by adding extra signing information: 
   + A validity period: for most CA’s (GlobalSign, VeriSign, …) the period is 1 year.  But for LetsEncrypt it is 3 months.
   + Digital signature: they create a hash of the certificate and encrypt that hash using their private key.  Afterwards any client can also calculate the hash, which should be the same as the hash from the CA combination of the public key and that metadata (a.o. the ‘common name’ which is the hostname of the server) is called a certificate.
3. The CA returns the certificate.

A hacker could sign his own certificate, but then he ends up with a self signed certificate.  Such certificates are not considered secure by most clients (e.g. browsers).

A hacker could create his own CSR for a domain (i.e. hostname) from somebody else.  However the CA will check whether the requestor of a certificate is also the real owner of that domain:
+ Most CA’s (GlobalSign, Verisign, …) will require that the CSR is being mailed from a special email adres in the same domain e.g. admin@same_domain.com, because then they are pretty sure that the requestor owns that domain.
+ LetsEncrypt offers free certificates via a full automated proces:
   1. Your acme client request a certificate from LetsEncrypt.
   2. LetsEncrypt returns a small challenge file.
   3. You need to make that file available on your domain (via port 80 or via a DNS record) within N minutes.
   4. When LetsEncrypt can find that file, they know you are owner of that domain (because only a root user can listen to port 80 or create DNS records).
   5. Then LetsEncrypt returns the certificate
