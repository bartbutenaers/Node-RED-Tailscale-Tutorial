# Tailscale for Node-RED
Tailscale is a mesh network between the Tailscale agents, which need to be installed on all the devices that need to be part of the virtual network. This *'tailnet'* will find automatically its way through the firewall on your modem/router, so ***no*** need to open ports or setup port forwarding:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/4b76fff8-b7a2-4b8b-8731-4cf78d88b7ce)

Two typical use cases for Node-RED that Tailscale secure:

+ *Access Node-RED dashboard/flow editor*: each device in the tailnet has a virtual ip address 100.x.y.z and a virtual hostname, both of which are only known within your tailnet.  When you navigate to such a virtual hostname in the browser of your phone, the Tailscale agent on your phone will forward your request to the Tailscale agent running on the specified virtual hostname (in this case your Raspberry).  That way you can access your Node-RED flow editor and/or dashboard from your smartphone via the internet.

+ *Access endpoints*: to allow the Google servers to send voice commands from your Google Home device to the node-red-contrib-smarthome node.  Since there is no possibility to install a Tailscale client on the Google Actions Console servers, those servers will not be able to access devices within your tailnet.  However you can setup a secure encrypted tunnel (called *'funnel'*) from your Node-RED port 3001 to the worldwide cluster of Tailscale Funnel servers.  That way you can expose a local service on the internet via your tailnet.  Afterwards you need to change the fulfillment callback url on the Google Actions server, to refer to your exposed service on the Tailscale Funnel servers.

## Wireguard traffic on 41641
All the encrypted Wireguard traffic between the Tailscale agents, will arrive on port 41641 by default.  The Tailscale agent will listen to traffic on that port, and forward it automatically to the port specified in the request.  For example:

1. You navigate in the browser of your smartphone to https://your-tailscale-machine-name.your-tailnet-name.ts.net:1880/ui
2. The Tailscale agent on your smartphone will send that request - via an encrypted Wireguard channel - to port 41641 of the target host.
3. The Tailscale agent will listen to that port 41641 and forward the request to port 1880 (as specified in the request).
4. Since Node-RED is listening to port 1880, it will handle it and serve the dashboard (as /ui is specified in the request).
5. In the browser of your smartphone, the Node-RED dashboard will appear.

So the tailnet is very transparent, since it looks like you navigate directly to port 1880 of the virtual host in your tailnet...
