# Use https in your tailnet
Previously we have deactivated https inside Node-RED, because we explained that the data within your tailnet is already encrypted (via Wireguard connections between 2 Tailscale agents).  As a result it is safe to use http connections within your tailnet, to access the Node-RED flow editor and dashboard.

In this section, we will start using https connections again! But as explained before, it is not advised to setup https *inside* the local Node-RED service.  Instead it is better to setup https in a reverse proxy *between* the internet and Node-RED.  Fortunately the Tailscale agents contain a reverse proxy.

## Tailnet: what goes in, will come out
It is ***not*** required to read this section.

The data between the Tailscale agents in your tailnet is already encrypted, using the Wireguard protocol used in the mesh network.  As a result all you data will be send in a secure way between your virtual devices in your tailnet, across the internet.  Which means it is already safe enough to access Node-RED via *plain http*, since all data is exchanged within your secure tailnet.

However it is still very useful to activate https, because the ***data is only encrypted between the Tailscale agents***!  Data that you send into a Tailscale agent will be encrypted by that agent, and the data will afterwards be decrypted by the other agent that receives that encrypted data.  As a result if you send unencrypted data (via a plain http connection) into your tailnet, it will leave your tailnet unencrypted.  What goes in will come out..

So the tailnet will simply pass your data transparent (without reading or changing the content), whether it is http or https traffic:

![image](https://github.com/user-attachments/assets/1f66b508-d180-4e14-8a8e-c55b9a6360c3)

In both cases the data will be encrypted inside your Tailnet via Wireguard.  But once the data comes out your tailnet, it will only be encrypted when https is being used.

## Why use https
There are a few reasons why you should setup a ***https*** connection through the already encrypted tailnet (which will result in double encryption inside the tailnet):
+ When you have very confidential data, and you don’t trust that the Tailscale claims their agent don't read your data (before encrypting it via Wireguard).
+ When the receiver expects https (based on certificates signed by a trusted CA).  For example the Google Action Console servers require such a https connection towards the node-red-contrib-google-smarthome node.
+ When web-push notifications are being used in the Node-RED dashboard, because modern browsers only allow such notifications when https is used (based on certificates signed by a trusted CA).
+ And so on ...

Anyway it is quite convenient if you can use https all over the place, even within your home network (i.e. within your LAN).

## Enable https in your tailnet
Previously we have activated DNS within our tailnet.  That is required to be able to use LetsEncrypt certificate, because the common name of the certificate should be the virtual hostname of your device.  It won't work for virtual ip addresses.

In the tabsheet *"DNS"* of your Tailscale admin console, you can now enable once https for your entire tailnet:

<img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/07aa2e7d-546f-443a-9801-cf9cc51b3156" width="500">

When https is enabled, the control servers will inform all your Tailscale agents that they are allowed to setup https connections based on LetsEncrypt certificates.  In other words the agents won't start using https immediately, they just get a notification that the are allowed to setup http connections.  The agents wait until they are instructed to setup https connections (using the `tailscale serve ...` or `tailscale funnel ...` commands as explained further on).

## Serve Node-RED via https (with LetsEncrypt certificates)
To setup a https connection to a local service (e.g. Node-RED), a certificate will be required.  Most users start by creating a ***self-signed certificate***, but they will soon realize that these certificates are nothing but trouble.  Indeed most systems (e.g. browsers, NodeJs, ...) don't trust such self-signed certificates, because even a hacker can create a self-signed certificate for your virtual domain/hostname. 

A much better solution is to start using certificates for your virtual domain/hostname, which are signed by a trusted certifcation authority (CA).  LetsEncrypt is a CA that offers certificated fully atomated and without any cost.  Most systems (e.g. browsers, NodeJs, ...) trust LetsEncrypt, so they accept https connections based on LetsEncrypt certificates.  Fortunately Tailscale agents can request LetsEncrypt certificates automatically.  The common name of those certificates, will be the virtual hostname of the device in the tailnet.  

To generate a LetsEncrypt certificate on our Raspberry Pi (where Node-RED is running), you might tell the Tailscale agent explicit to generate a LetsEncrypt certificate, using the command `tailscale cert your-machine-name.your-tailnet-name.ts.net`.  However we will ***NOT*** do it like that.  Because it is even more easier to just tell the agent that you need a https connection, for https requests on a specified port. Then the agent will automatically take care of everything.

Execute the following command (on the device running Node-RED), to ***'serve'*** the local Node-RED (flow editor) service via https:
```
tailscale serve --https=443 --bg --set-path /flow_editor http://localhost:1880/flow_editor
```
Some explanation about the command line parameters:
+ Listen to https requests on port 443, and start https connection (and as a result create and renew a LetsEncrypt certificate).  You could use any free port number, but port 443 is the default https port.  So you don't need to specify it in the address bar of your browser, which simplifies the flow editor url.  For example:
   ```
   https://your-device-virtual-ip-address/flow_editor
   ```
   This url will behind the scenes converted automatically by your browser to:
   ```
   https://your-device-virtual-ip-address:443/flow_editor
   ```
+ Run as a background (bg) job, to make sure the serving keeps running when the command shell window is closed.
+ The httpAdminRoot path is used here `/flow_editor` (from the settings.js file).
+ All requests will be forwarded to port 1880 on localhost (sub-path 'flow_editor'), to which Node-RED is listening (to plain http connections).

Via the above command you tell the reverse proxy (of the Tailscale agent on the Raspberry) to listen to port 443 via a https server, and forward these as http requests to port 1880 on localhost.  The local (Node-RED) service will now become this way ***only accessible within the tailnet*** via https.

Remarks:
+ Node-RED is also still accessible directly via http on port 1880.
+ It is absolutely required to specify a port number (e.g. 443), otherwise you cannot combine 'serve' and 'funnel' (see my [issue](https://github.com/tailscale/tailscale/issues/11009#issuecomment-2267159080)).
+ Currently requests can only be forwarded by the reverse proxy to localhost, not to other hostnames.
+ Via `tailscale serve reset` it is possible to remove all current serve's from the reverse proxy.
+ Via `tailscale serve status` you can get a list of all current serve's that have been setup, but it will also show all available funnels (which will be discussed later on).
+ Via the above command we serve a local service via a reverse proxy, however it is also possible to serve files and static texts on your tailnet easily.  This is possible via respectively a file server and a static text server, which are available within the Tailscale agent (see Tailscale [documentation](https://tailscale.com/kb/1242/tailscale-serve)). 

## Check the certificate
Once the reverse proxy has been setup to use https, we can have a look at the certificate that is being used.

1. Navigate to Node-RED in the browser:
   ```
   https://your-machine-name.your-tailnet-name.ts.net:9123/dashboard
   ```
2. As soon as the browser shows in the address bar that the connection is safe, that already means that a valid LetsEncrypt certificate is being used.
3. Depending on the browser, there will be a menu option available to show the certificate being used.  Via that way you should be able to see the LetsEncrypt certificate for your virtual hostname.  For example in Chrome:

   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/e9772288-9ddd-4168-9635-fa816ed9cdbd)

## Behind the scenes
It is ***not*** required to read this section.  This section explains in detail what happens behind the scenes if a local service is being served:

![image](https://github.com/user-attachments/assets/b712d880-563d-4dbe-a921-f1cef4752df6)

1. Execute the `tailscale serve ...` command, to configure the reverse proxy inside the Tailscale agent.
2. The agent sends a request to the Tailscale Control servers, to get a LetsEncrypt certifcate (for the domain/hostname *"your-machine-name.your-tailnet-name.ts.net"*).
3. The Tailscale Control servers forward the request to the LetsEncrypt service, which will check if Tailscale owns the public domain *"your-machine-name.your-tailnet-name.ts.net"*.
   
   Remark: The Tailscale control servers will create a *.ts.net DNS TXT record for your subdomain, to complete their DNS-01 challenges for LetsEncrypt. 
4. Since that is the case (because tailscale owns all the subdomains of *.ts.net), the LetsEncrypt service will return a LetsEncrypt certificate to the Tailscale Control servers.
5. The Tailscale control servers forward the LetsEncrypt certificates to the Tailscale agent.
6. The Tailscale agent will store the certificate in the */var/lib/tailscale/certs* directory (on Linux), next to the corresponding private key file.
7. The Tailscale agent reverse proxy will load the LetsEncrypt certificate and private key, which it can use to setup SSL connections when a https request arrives.
8. When you navigate in your browser to the virtual hostname of the Raspberry Pi (https://your-machine-name.your-tailnet-name.ts.net:9123/dashboard), the Tailscale agent on your smartphone will intercept that request.
9. The DNS resolver of the agent will detect that it is a virtual Tailscale ip address or hostname, so it will forward the request to the Tailscale agent (on the Raspberry Pi) in your tailnet. 
10. The Tailscale agent listens to requests arriving via the Wireguard mesh network on port 4161 by default.  The agent will forward the https request to the port specified inside the request, in this case port 9123.
11. The reverse proxy will now execute 3 tasks:
     + Setup a https connection with the smartphone, based on the LetsEncrypt certificate.
     + Do SSL termination (i.e. convert the https requests to http requests) because we have specified in our serve command that the target is http.
     + Forward the http request to port 1880.  
12. Finally the plain http request will be forwarded to Node-RED, which will show the dashboard.

Since LetsEncrypt certificates only have a validity period of 3 months, the Tailscale agent will automatically ***renew*** these certificates periodically.  You can check that by looking at the date of the .crt file in the */var/lib/tailscale/certs* directory.

## SSL termination
Note that there are two kind of connections being used:
+ A https connection between the browser and the Tailscale agent on the Raspberry.  
+ A plain http connection between the agent and Node-RED.

It would be quite useles to setup a second https connection between the Tailscale agent (on the Raspberry) and Node-RED, because:
+ All the data traffic stays inside your own Raspberry.
+ If you have setup in the past https in Node-RED using a self-signed certificate, the Tailscale agent might even refuse to connect to Node-RED, because it doesn't trust that certificate.
