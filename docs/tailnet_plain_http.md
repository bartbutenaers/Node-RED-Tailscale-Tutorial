# Access Node-RED via plain http

In the previous part we have setup a tailnet, and one of its virtual devices is a Raspberry Pi running Node-RED.  Now we will try to access the Node-RED dashboard (via plain http) from another virtual device inside our tailnet.  For example access the dashboard on an Android smartphone.

All Tailscale agents are connected to each other using a mesh network.  All those connections between the Tailscale agents are encrypted via Wireguard:

![image](https://github.com/user-attachments/assets/8bc42906-bfaa-4f15-868a-17a04256718a)

The Tailscale agents use Wireguard to encrypt all the data being send, because your device (running a Tailscale agent) might be located on the internet.  And you don't want other people to be able to intercept your data.  As a result it is secure to use http connections ***within*** your Tailnet, because only your devices (running a Tailscale agent) have access to those http connections.

## Disable https in Node-RED!
When you have previously setup https in Node-RED, it is advised to turn it off.  That can be achieved by putting the following statements (in settings.js file) in comment, and restart Node-RED:
```
   requireHttps: false,
   //https: function() {
   //   return {
   //      key: require("fs").readFileSync('xxx.key'),
   //      cert: require("fs").readFileSync('yyy.crt')
   //   }
   //},
```
Of course all other stuff related to this (e.g. my node-red-contrib-letsencrypt node) can also be removed.

It might sound very unsecure to turn off https in Node-RED, but we have a few reasons to do this:
+ It is not advised to setup security *inside* the (Node-RED) application, because then it can be turned of when a malafide user gains access to the application.  Instead take care of this *before* the application, i.e. in the Tailscale agent.
+ Lots of users use *self-signed certificates* which cause a lot of troubles, while the Tailscale agent offers LetsEncrypt certificates.
+ Having to setup https in every application results in a rather complex system, while it is easier to maintain when the Tailscale agent can offer this to all applications on the same host.

Setting up https in the Tailscale agent will be described later on in this tutorial.

## Basic authentication
Because only devices from your tailnet will be able to connect to your Node-RED system, you could turn off basic authentication in Node-RED.  However since Node-RED is a rather critical system in your home automation, it might still be better to enable basic authentication.  That way it is required to enter username and password in the logon screen, as an extra layer of protection.

Basic authentication can be configured in the Node-RED settings.js file:
```
adminAuth: {
   "type": "credentials",
   "users": [ { "username": "zzz", "password": "xyz", "permissions": "*" } ]
},
```
Note that the password needs to be hashed, before entering it in the above file.  You can do that using following [command](https://nodered.org/docs/user-guide/runtime/securing-node-red#generating-the-password-hash):
```
node-red admin hash-pw
```

## Setup a base url
Later on, we need to be able to run all applictions at their own base url.  So we need to fix this also for Node-RED.

+ The new Node-RED *dashboard* is by default available at base url `/dashboard`.  So that is ok already.
+ The Node-RED *flow editor* is by default available at base url `/`.  You can specify a custom flow editor base url in the settings.js file (and restart Node-RED), for example:
   ```
   httpAdminRoot: '/flow_editor',
   ```

## Access the local services via http
On every device in your tailnet, there might be one or more local services running (which are listening to a port).  For example the Node-RED service is listening by default to port 1880.

Be aware that ***ALL the local services become accessible*** in your tailnet, via the Tailscale agent!  In most cases that shouldn't be a problem, because your tailnet only contains trusted devices which are allowed to access your services.  However if you don't want that, you can find information later on how you can limit which ports are accesible via your tailnet (see section about Access Control).

Before setting up https, we will first try to access the Node-RED flow editor and dashboard via http from any device in your tailnet.  You can do that by navigating to following address in the browser:
```
http://your-device-virtual-ip-address:1880/flow_editor
http://your-device-virtual-ip-address:1880/dashboard
```
Note: You can find the virtual ip address (of your device running Node-RED) in the *"Machines"* tabsheet on your Tailscale account.

## How this works behind the scenes
It is ***not*** required to read this detailed information, but it helps to gain insight in your setup.

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

7. Note that you can also navigate directly to Node-RED, via the *physical ip address or hostname* of the device:
   ```
   http://<your-physical-device-ip-address>:1880/dashboard
   ```
   However it is ***NOT*** advised to use that, because it will only work when both devices (i.e. smartphone and Raspberry Pi) are within your LAN.  When one of the devices is not insde your LAN, your modem/router/firewall will block the request (since port forwarding has been disabled previously).  On the other hand the url using the tailnet (see step 1) will work wherever your both devices are located.
