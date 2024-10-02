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
To setup https, a certificate will be required.  Most users start by creating a ***self-signed certificate***, but they soon figure out that these certificates are nothing but trouble.  Because most systems (e.g. browsers, NodeJs, ...) don't trust such self-signed certificates, because even a hacker can create a self-signed certificate for your domain/hostname. 

A much better solution is to start using certificates for your domain/hostname, which are signed by LetsEncrypt.  Most systems (e.g. browsers, NodeJs, ...) trust LetsEncrypt, so they accept your https connection.  Fortunately Tailscale agents can request LetsEncrypt certificates automatically (with common name the virtual hostname of the device in the tailnet).  Thanks to that it now becomes even more easy to use LetsEncrypt certificates, compared to create self-signed certificates.

In our case we want a https connection to Node-RED, which means we will need to generate a LetsEncrypt certificate on our Raspberry Pi where Node-RED is running.

It is possible to ask a Tailscale agent to generate a LetsEncrypt certificate, using the command `tailscale cert your-machine-name.your-tailnet-name.ts.net`.  However it is much easier to just tell the agent that you need a https connection on a specified port, and then the agent will automatically take care of everything:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/317d052a-4935-44ff-b063-5d83c0849843)

1. Execute the command `tailscale funnel --https=443 ...` to expose a local service respectively public on the internet, or `tailscale serve --https=9123 ...` to expose that service private within your tailnet.

2. In both cases, the Tailscale agent knows that it needs to send a request to the Tailscale Control servers, that it requires a LetsEncrypt certifcate (for the domain/hostname *"your-machine-name.your-tailnet-name.ts.net"*).

3. The Tailscale Control servers forward the request to the LetsEncrypt servers, which return - after doing the necessary checks - a LetsEncrypt certificate to the Tailscale Control servers.  This is all possible because Tailscale owns the public domain *"your-machine-name.your-tailnet-name.ts.net"*.

4. The Tailscale control servers forward the LetsEncrypt certificates to the Tailscale agent which stores the certificate in the */var/lib/tailscale/certs* folder (on Linux), next to the corresponding private key file.

5. The Tailscale agent will automatically ***renew*** these certificates periodically, to make sure you will never end up with a broken https connection (due to an expired LetsEncrypt certificate).  Which is necessary because LetsEncrypt certificates have a validity period of 3 months.

6. As soon as an https connection handshake is started on the specified ports (in this example ports 443 and 9123), the LetsEncrypt certificate will be used automatically by the reverse proxy inside the Tailscale agent.

7. And voila, you have a https connection to your Node-RED system.

## Setup a https connection
A this point you have LetsEncrypt certificates generated, but they are ***NOT*** being used yet!
Now it is time to setup a https connection, based on this LetsEncrypt certificate.

In a next tutorial we will setup a Caddy reverse proxy to use the certificate to setup http connections.  But at this moment we are going to keep it simple, and let Node-RED use it to setup http connections.  Note that is not optimal because if a hacker gains access to Node-RED, he will also be able to see your private key!  Keep that in mind if you want to keep doing it that way...

1. Since both files (cert and key) will have owner and group ‘root’, Node-RED cannot read their content (since Node-RED does not run as root user).  So you need e.g. copy both files e.g. to your .node-red folder an adjust the owner an group of the files to the Node-RED user.
2. Adjust the Node-RED settings.js file to start using this new key pair:
   ```
   requireHttps: true,
   https: function() {
      return {
         key: require("fs").readFileSync('/path/to/some/folder/your_machine_name.your_tailnet_name.ts.net.key'),
         cert: require("fs").readFileSync('/path/to/some/folder/your_machine_name.your_tailnet_name.ts.net.crt')
      }
   },
   ```
3. Restart Node-RED to activate the updated https settings.

Now test this setup, by navigating to your flow editor https://your-machine-name.your-tailnet-name.ts.net:1880/http-admin-root-path-if-available.
If the certifcate has been used, your browser should let you know that the ***connection is secure***.
