# Access via plain http

In the previous part we have setup a tailnet, and one of its virtual devices is a Raspberry Pi running Node-RED.  Now we will try to access the Node-RED dashboard (via plain http) from another virtual device inside our tailnet.  For example access the dashboard on an Android smartphone.

All Tailscale agents are connected to each other using a mesh network.  All those connections between the Tailscale agents are encrypted via Wireguard:

![image](https://github.com/user-attachments/assets/8bc42906-bfaa-4f15-868a-17a04256718a)

They use Wireguard to encrypt all the data being send, because your device (running a Tailscale agent) might be located on the internet.  And you don't want other people to be able to intercept your data.  As a result it is secure to use (insecure) http connections ***within*** your Tailnet, because only your devices (running a Tailscale agent) have access to those http connections.

## Disable https in Node-RED!
This step is only required if you had previously setup https in Node-RED.  For example using a self-signed certificate, or using the node-red-contrib-letsencrypt node.  Because from here on we will ***disable https in Node-RED!***  This might sound very weird and unsecure, but it will be explained in a the https section afterwards.

If you have setup https in Node-RED in the past, you need to remove that config again from your settings.js file.  So put the following statements in comment, and restart Node-RED:
```
   requireHttps: false,
   //https: function() {
   //   return {
   //      key: require("fs").readFileSync('/path/to/some/folder/your_machine_name.your_tailnet_name.ts.net.key'),
   //      cert: require("fs").readFileSync('/path/to/some/folder/your_machine_name.your_tailnet_name.ts.net.crt')
   //   }
   //},
```

## Test: Access a local service
Try to access your Node-RED flow editor from any device (running a Tailscale agent), by navigating to following address in the browser:
`http://your-device-virtual-ip-address:1880/`

Now the Node-RED flow editor should appear, via a plain http connection within your tailnet.  Note that you can find the virtual ip address (of your device running Node-RED) in the *"Machines"* tabsheet on your Tailscale account.

If you have setup following option in your Node-RED settings.js file:
```
httpAdminRoot: '/my_flow_editor',
```
Then your flow editor will be available at `http://your-device-virtual-ip-address:1880/my_flow_editor`.

That way you have a sub-path both for the dashboard and the flow editor, which is a bit more organised.  Moreover that way it becomes more difficult for hackers to guess the correct url.

## How this works behind the scenes
It is ***not*** required to read this detailed information, but it helps to gain insight in your setup.

On your virtual devices there might be one or more local services running, which are listening to ports.  For example the Node-RED service is listening by default to port 1880.  Be aware that by default ***ALL*** local services become accessible via the Tailscale agent to all your devices in your tailnet!  In most cases that shouldn't be a problem, because your tailnet only contains trusted devices which are allowed to access your services.  However if you don't want that, you can find information later on how you can limit which ports are accesible via your tailnet (see section about Access Control).

![image](https://github.com/user-attachments/assets/94ce55a6-03f6-4be5-a027-a2a69204d443)

1. Navigate on any device (running a Tailscale agent) to the Node-RED dashboard, by entering the url in your browser on your smartphone:

   `http://your-device-virtual-ip-address:1880/dashboard`

   Note that afterwards we will show how to setup DNS in tailscale, so you can navigate to a virtual hostname (instead of a virtual IP address).
2. The http request will be intercepted by the Tailscale agent ***DNS resolver***, which will determine whether the destination is a Tailscale virtual ip address (or hostname).
   
   Note: the devices in your tailnet use their local DNS settings and only use the tailnet's DNS servers when needed.  For example when a virtual Tailscale IP address or hostname is being pinged, that request will be forwarded to your tailnet.
3. If the destination is a Tailscale device, the Tailscale agent will forward the http request - via your tailnet (i.e. via an encrypted wireguard channel) - to the Tailscale agent on the destination device.  For example to your Raspberry Pi, running Node-RED.
4. The Tailscale agent (on the destination device) will listen to port 41641 by default, for all incoming requests arriving from the Wireguard mesh network.  Since the http requests contains target port 1880, the ***reverse proxy*** (in the Tailscale agent) will forward the request to that port 1880.
   
   Note: the reverse proxy in the Tailscale agent has both a http server and a https server, which will respectively handle http and https connections.  The latter one will be discussed later on, when we `serve` local services via https.
5. The http request will be sent to Node-RED, which is listening to port 1880.
6. Node-RED will return a http response. After the http response has travelled the entire traject backwards, the Node-RED dashboard should appear in the browser (via plain http):

   ![image](https://github.com/user-attachments/assets/df585c26-46a8-421a-a2b7-183733f560fd)

7. Note that you can still navigate directly to Node-RED, via the physical ip address (or hostname) of the device: http://<your-physical-device-ip-address>:1880/dashboard
   
   But that will only work when both devices (i.e. smartphone and Raspberry Pi) are within your LAN, because otherwise your modem/router/firewall will block the request (since port forwarding has been disabled previously).  While the url via your tailnet will work whatever the location of your both devices.

It should now already become a bit more clear why we have asked above to disable https inside Node-RED:
+ It has not much use to use https between the Tailscale agent and Node-RED, because the data stays inside your Raspberry Pi which is considered secure.
+ If you have used a self sign certificate to setup https connections inside Node-RED, the Tailscale agent will not trust it and reject the https connection.
