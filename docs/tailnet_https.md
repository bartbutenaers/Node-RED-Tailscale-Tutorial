# Use https in your tailnet
The data between the Tailscale agents in your tailnet is already encrypted, using the Wireguard protocol.  As a result all you data will be send in a secure way between your virtual devices in your tailnet, across the internet.  Which means it is already safe enough to get started, without any need to activate https!  In other words you can use ***plain http*** to access Node-RED, since all data is exchanged within your secure tailnet.

However it is still very useful to activate https, because the data is only encrypted ***between*** the Tailscale agents.  Data that you send into a Tailscale agent will be encrypted by that agent, and the data will be decrypted by the agent that receives that encrypted data.  As a result if you send unencrypted data into your tailnet, it will leave your tailnet unencrypted.  What goes in will come out..

So the tailnet will simply pass your data transparent (without reading or changing the content), whether it is http or https traffic:

![image](https://github.com/user-attachments/assets/1f66b508-d180-4e14-8a8e-c55b9a6360c3)

## Why use https
There are a few reasons why you should setup a ***https*** connection through the encrypted tailnet:
+ When you have very confidential data, and you don’t trust that e.g. the Tailscale agents don't read the data (before encrypting it via Wireguard).
+ When the receiver expects https (based on signed certificates), like for example the Google Action Console servers (to receive voice commands from a Google Home device).
+ When web-push notifications are being used in the Node-RED dashboard, because modern browsers only allow such notifications when https is used (based on signed certificates like e.g. LetsEncrypt).
+ And so on ...

Anyway it is quite convenient if you can use https all over the place, even within your home network (i.e. within your LAN).

## Enable https in your tailnet
In the tabsheet *"DNS"* of your Tailscale admin console, you can enable https once for your entire tailnet:

<img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/07aa2e7d-546f-443a-9801-cf9cc51b3156" width="500">

When https is enabled, the control servers will inform all your Tailscale agents that they are allowed to setup https connections based on LetsEncrypt certificates.  Note that the agents won't start using https immediately, they just wait until they are told to do so (using the `tailscale serve ...` or `tailscale funnel ...` commands as explained further on).

## Use LetsEncrypt certificates
To setup a https connection to a local service (e.g. Node-RED), a certificate will be required.  Most users start by creating a ***self-signed certificate***, but they will soon realize that these certificates are nothing but trouble.  Indeed most systems (e.g. browsers, NodeJs, ...) don't trust such self-signed certificates, because even a hacker can create a self-signed certificate for your domain/hostname. 

A much better solution is to start using certificates for your domain/hostname, which are signed by a trusted certifcation authority (CA).  LetsEncrypt is a CA that offers certificated fully atomated and without any cost.  Most systems (e.g. browsers, NodeJs, ...) trust LetsEncrypt, so they accept such https connections.  Fortunately Tailscale agents can request LetsEncrypt certificates automatically (with common name the virtual hostname of the device in the tailnet).  

To generate a LetsEncrypt certificate on our Raspberry Pi (where Node-RED is running), you might tell the Tailscale agent to generate a LetsEncrypt certificate, using the command `tailscale cert your-machine-name.your-tailnet-name.ts.net`.  However we will ***NOT*** do it like that.  Because it is even more easier to just tell the agent that you need a https connection on a specified port, and then the agent will automatically take care of everything:

![image](https://github.com/user-attachments/assets/bdad301b-bb0a-488a-8e7f-7c85af0ddb4b)

1. Execute the following command to serve the local Node-RED service ***within your tailnet only*** via https:
   ```
   tailscale serve --https=9123 --bg --set-path /my_serve http://localhost:1880
   ```
   That way you tell the reverse proxy (of the Tailscale agent on the Raspberry) to listen to port 9123 via a https server, and forward these as http requests to port 1880 on localhost.

   Remarks:
   + The port 9123 is random unused port that I have choosen.
   + It is absolutely required to specify a port number, otherwise you cannot combine 'serve' and 'funnel' (see my [issue](https://github.com/tailscale/tailscale/issues/11009#issuecomment-2267159080)).
   + Currently requests can only be forwarded to localhost, not to other hostnames.
3. The agent sends a request to the Tailscale Control servers, to mention that it requires a LetsEncrypt certifcate (for the domain/hostname *"your-machine-name.your-tailnet-name.ts.net"*).
4. The Tailscale Control servers forward the request to the LetsEncrypt service, which will check if Tailscale owns the public domain *"your-machine-name.your-tailnet-name.ts.net"*.
5. Since that is the case, the LetsEncrypt service will return a LetsEncrypt certificate to the Tailscale Control servers.
6. The Tailscale control servers forward the LetsEncrypt certificates to the Tailscale agent.
7. The Tailscale agent will store the certificate in the */var/lib/tailscale/certs* directory (on Linux), next to the corresponding private key file.
8. The Tailscale agent reverse proxy will load the LetsEncrypt certificate and private key.
9. When you navigate in your browser to the virtual hostname of the Raspberry Pi (https://your-machine-name.your-tailnet-name.ts.net), the Tailscale agent will intercept that request.
10. The DNS resolver of the agent will detect that it is a virtual Tailscale ip address or hostname, so it will forward the request to the Tailscale agent (on the Raspberry Pi) in your tailnet.
11. As soon as a http(s) request arrives on one of the specified https ports (in this example ports 443 or 9123), there will be an SSL handshake between the browser and the Tailscale agent (based on the LetsEncrypt certificate and private key).  The LetsEncrypt certificate will be used by the reverse proxy to setup the https connection.
12. Finally the request will be forwarded to Node-RED (via plain http).

Since LetsEncrypt certificates only have a validity period of 3 months, the Tailscale agent will automatically ***renew*** these certificates periodically.  You can check that by looking at the date of the .crt file in the */var/lib/tailscale/certs* directory.

## SSL termination
Note that there are two kind of connections being used:
+ A https connection between the browser and the Tailscale agent on the Raspberry.  
+ A plain http connection between the agent and Node-RED.

It would be quite useles to setup a second https connection between the Tailscale agent (on the Raspberry) and Node-RED, because:
+ All the data traffic stays inside your own Raspberry.
+ If you have setup in the past https in Node-RED using a self-signed certificate, the Tailscale agent might even refuse to connect to Node-RED, because it doesn't trust that certificate.
