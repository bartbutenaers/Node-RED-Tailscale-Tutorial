## Tailscale control servers
Tailscale offers a worldwide cluster of control servers which have multiple tasks:
+ DHCP server to assign virtual IP addresses (100.x.y.z) to Tailscale agents in your tailnet.
+ Host the admin console, where you can manage your tailnet.  
   Note: Every time you want to access the admin console, you will need to authenticate via your identity provider.
+ Store the public LetsEncrypt certificates for all Tailnet agents, and distribute those to the other agents in your tailnet.
+ And so on...

The following diagram explains how you can control the devices in your tailnet via the Tailscale control servers:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/972b8bf2-c860-444c-8961-5d75d7a5359c)

Since portforwarding is not being used, the Tailscale agent need to find a way to communicate with both the Tailscale servers and the other devices in your tailnet.  Which is not a simple task.  One of the requirements to make this work, is that the ***Tailscale agent is allowed to listen to standard https port 443***!  So if you have already another service (e.g. Node-RED) listening to that port, make sure to fix that!  Otherwise the tailscaled daemon on your Raspberry Pi will log an error during startup, that another service is already listening to that port...

You can access your tailnet configation via the ***admin interface*** on the control servers:
1. Navigate to the control servers via https://login.tailscale.com/admin
2. Logon to your tailnet via your identity provider
3. In the next sections we will configure our tailnet.
