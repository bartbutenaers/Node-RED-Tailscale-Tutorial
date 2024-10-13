# Access via plain http

In the previous part we have setup a tailnet, and one of its virtual devices is a Raspberry Pi running Node-RED.  Now we will try to access the Node-RED dashboard from another virtual device inside our tailnet, for example an Android smartphone.

## Disable https in Node-RED
This step is only required if you had previously setup https in Node-RED.  For example using a self-signed certificate, or using the node-red-contrib-letsencrypt node.  Because from here on we will disable https in Node-RED!  This might sound very weird and unsecure, but it will be explained in a the https section afterwards.

If you have setup https in Node-RED in the past, you need to remove that config again from your settings.js file (and restart Node-RED):
```
   requireHttps: false,
   //https: function() {
   //   return {
   //      key: require("fs").readFileSync('/path/to/some/folder/your_machine_name.your_tailnet_name.ts.net.key'),
   //      cert: require("fs").readFileSync('/path/to/some/folder/your_machine_name.your_tailnet_name.ts.net.crt')
   //   }
   //},
```

## Access a local service
On our virtual devices there might be one or more local services be running, and listening to ports.  For example the Node-RED service is listening by default to port 1880.  Be aware that by default ***ALL*** local services are accessible via the Tailscale agent on your tailnet!  Normally that shouldn't be a problem, because your tailnet only contains trusted devices which can access your services.  However if you don't want that, you can find information further on how you can limit which ports are accesible via your tailnet (see section about Access Control).

![image](https://github.com/user-attachments/assets/a415a914-4f76-4f45-9a49-e73635356928)

1. Enter the virtual IP address of your Raspberry Pi in the browser on your smartphone, and navigate to port 1880 (and subpath 'dashboard'):

   `http://your-device-virtual-ip-address:1880/dashboard`

   Note that afterwards we will show how to setup DNS in tailscale, so you can navigate to a hostname (instead of an IP address).
2. The http request will be intercepted by the Tailscale agent ***DNS resolver***, which will detect that it is a Tailscale virtual ip address (or hostname).
   Note that the devices in your tailnet use their local DNS settings and only use the tailnet's DNS servers when needed.  For example when a virtual Tailscale IP address or hostname is being pinged, that request will be forwarded to your tailnet.
3. The http request will be send via the tailnet (i.e. via an encrypted wireguard channel) to the tailscale on the Raspberry Pi.
4. The Tailscale agent will listen to port 41641 by default for all requests arriving from the Wireguard mesh network.  Since the http requests contains target port 1880, the ***reverse proxy*** will forward the request to that port.
   Note: the reverse proxy in the Tailscale agent has both a http server and a https server, which will respectively handle http and https connections.  The latter one will be discussed later on, when we `serve` local services via https.
5. The http request will be sent to Node-RED, which will return a http response.
6. After the http response has travelled the entire traject, the Node-RED dashboard should appear in the browser:

   ![image](https://github.com/user-attachments/assets/df585c26-46a8-421a-a2b7-183733f560fd)

## Path to flow editor

By default your flow editor will become available at `http://your-device-virtual-ip-address:1880`.  Optionally the flow editor can be made accessible at a sub-path, for example at `http://your-device-virtual-ip-address:1880/my_flow_editor`, by adjusting following property in your Node-RED settings.js file:
```
httpAdminRoot: '/my_flow_editor',
```
That way you have both sub-paths for the dashboard and flow editor, which is a bit more organised.  Moreover it becomes more difficult for hackers to guess the correct url.
