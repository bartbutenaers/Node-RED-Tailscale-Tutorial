# Access via plain http

In the previous part we have setup a tailnet, and one of its virtual devices is a Raspberry Pi running Node-RED.  Now we will try to access the Node-RED dashboard from another virtual device inside our tailnet, for example an Android smartphone.

## Disable https in Node-RED
This step is only required if you had previously setup https in Node-RED.  For example using a self-signed certificate, or using the node-red-contrib-letsencrypt node.  Because from here on we will disable https in Node-RED!  This might sound unsecure, but it will be explained in a next part.

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
On our virtual devices there might be one or more local services be running, and listening to ports.  For example the Node-RED service is listening by default to port 1880.  Be aware that by default ***ALL*** local services are accessible via your tailnet!  If you don't want that, you can find information afterwards how you can limit which ports are accesible via your tailnet (see section about Access Control).

![image](https://github.com/user-attachments/assets/ccdfb8ec-c247-4b4a-856e-7bd857db3076)

1. Enter the virtual IP address of your Raspberry Pi in the browser on your smartphone, and navigate to port 1880 (and subpath 'dashboard'):

   `http://your-device-virtual-ip-address:1880/dashboard`

   Note that afterwards we will show how to setup DNS in tailscale, so you can navigate to a hostname (instead of an IP address).
2. The http request will be intercepted by the Tailscale agent ***DNS resolver***, which will detect that it is a Tailscale virtual ip address (or hostname).
3. The http request will be send via the tailnet to the tailscale on the Raspberry Pi.
4. The ***reverse proxy*** in the tailscale agent will forward the request to port 1880.
5. The http request will be sent to Node-RED, and the http response will be returned.
6. After the http response has travelled the entire traject, the Node-RED dashboard should appear in the browser:

   ![image](https://github.com/user-attachments/assets/df585c26-46a8-421a-a2b7-183733f560fd)

