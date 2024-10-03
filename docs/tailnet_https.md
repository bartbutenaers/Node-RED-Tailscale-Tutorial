# Use https in your tailnet
The data between the Tailscale agents in your tailnet is already encrypted, using the Wireguard protocol.  As a result all you data will be send in a secure way between your virtual devices in your tailnet, across the internet.  Which means it is already safe enough to get started, without any need to activate https!

However it is still useful to activate https, because the data is only encrypted ***between*** the Tailscale agents.  Data that you send into a Tailscale agent will be encrypted by that agent, and the data will be decrypted by the agent that receives that encrypted data.  As a result if you send unencrypted data into your tailnet, it will leave your tailnet unencrypted.  What goes in will come out..

So the tailnet will simply pass your data transparent (without reading or changing the content), whether it is http or https traffic:

![image](https://github.com/user-attachments/assets/1f66b508-d180-4e14-8a8e-c55b9a6360c3)

There are a few reasons why you should setup a ***https*** connection through the encrypted tailnet:
+ When you have very confidential data, and you don’t trust that e.g. the Tailscale agents don't read the data (before encrypting it via Wireguard).
+ When the receiver expects https (based on signed certificates), like for example the Google Action Console servers (to receive voice commands from a Google Home device).
+ When web-push notifications are being used in the Node-RED dashboard, because modern browsers only allow such notifications when https is used (based on signed certificates like e.g. LetsEncrypt).
+ And so on ...

Anyway it would be handy overall if you can use https all over the place, even within your home network (i.e. within your LAN).

## Enable https
In the tabsheet *"DNS"* of your Tailscale admin console, you can once enable https:

<img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/07aa2e7d-546f-443a-9801-cf9cc51b3156" width="500">

When https is enabled, the control servers will inform all your Tailscale agents that they are allowed to setup https connections based on LetsEncrypt certificates.

## Using LetsEncrypt certificates
To setup https, a certificate will be required.  Most users start by creating a ***self-signed certificate***, but they soon realize that these certificates are nothing but trouble.  Indeed most systems (e.g. browsers, NodeJs, ...) don't trust such self-signed certificates, because even a hacker can create a self-signed certificate for your domain/hostname. 

A much better solution is to start using certificates for your domain/hostname, which are signed by LetsEncrypt.  Most systems (e.g. browsers, NodeJs, ...) trust LetsEncrypt, so they accept your https connection.  Fortunately Tailscale agents can request LetsEncrypt certificates automatically (with common name the virtual hostname of the device in the tailnet).  Thanks to that it now becomes even more easy to use LetsEncrypt certificates, compared to create self-signed certificates.

In our case we want a https connection to Node-RED, which means we will need to generate a LetsEncrypt certificate on our Raspberry Pi where Node-RED is running.

It is possible to ask a Tailscale agent to generate a LetsEncrypt certificate, using the command `tailscale cert your-machine-name.your-tailnet-name.ts.net`.  However it is much easier to just tell the agent that you need a https connection on a specified port, and then the agent will automatically take care of everything:

![image](https://github.com/user-attachments/assets/b9cf102e-7f7d-47bf-a9c7-33057dc4cf67)

1. Execute in the CLI the below command to expose the local Node-RED service private within your tailnet only.  That way the devices within your tailnet can access Node-RED, but it is not public available to the outside world:
   ```
   tailscale serve --https=9123 --bg --set-path /my_serve http://localhost:1880
   ```
   Based on the second parameter, the Tailscale agent knows that the connection requires https.
   
   Note that the port 9123 is a random port number at your choice, but it must be a free port (on which no application is listening).  It took me a while to figure that out (see [here](https://github.com/tailscale/tailscale/issues/11009#issuecomment-2267159080)).
3. The agent sends a request to the Tailscale Control servers, that it requires a LetsEncrypt certifcate (for the domain/hostname *"your-machine-name.your-tailnet-name.ts.net"*).
4. The Tailscale Control servers forward the request to the LetsEncrypt service, which will check if Tailscale owns the public domain *"your-machine-name.your-tailnet-name.ts.net"*.
5. Since that is the case, the LetsEncrypt service will return a LetsEncrypt certificate to the Tailscale Control servers.
6. The Tailscale control servers forward the LetsEncrypt certificates to the Tailscale agent.
7. The Tailscale agent will store the certificate in the */var/lib/tailscale/certs* directory (on Linux), next to the corresponding private key file.
8. The Tailscale agent reverse proxy will load the LetsEncrypt certificate and private key.
9. When you navigate in your browser to the virtual hostname of the Raspberry Pi (https://your-machine-name.your-tailnet-name.ts.net), the Tailscale agent will intercept that request.
10. The DNS resolver of the agent will see that it is a virtual Tailscale hostname, so it will forward the request to the Tailscale agent (on the Raspberry Pi) in your tailnet.
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

So if you have setup https in Node-RED in the past, you need to remove that config again from your settings.js file (and restart Node-RED):
```
   requireHttps: false,
   //https: function() {
   //   return {
   //      key: require("fs").readFileSync('/path/to/some/folder/your_machine_name.your_tailnet_name.ts.net.key'),
   //      cert: require("fs").readFileSync('/path/to/some/folder/your_machine_name.your_tailnet_name.ts.net.crt')
   //   }
   //},
```
