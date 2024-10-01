## Tailscale control servers
Tailscale offers a worldwide cluster of control servers which perform multiple tasks:
+ They offer a ***DHCP server*** to assign virtual IP addresses (100.x.y.z) to new Tailscale agents in your tailnet.
+ They host the ***admin console***, where you can manage your tailnet .  The admin console is accessible via https://login.tailscale.com/start  
   Note: Every time you want to access the admin console, you will need to authenticate yourself via your preferred identity provider (Google, Microsoft, Github, ...).
+ The offer a ***DNS server*** to assign virtual hostnames to Tailscale agents in your tailnet, at least if you have enabled DNS in your tailnet.
+ They store the public ***LetsEncrypt certificates*** for all Tailnet agents, and distribute those certificates to all the agents in your tailnet.
+ And so on...

So you setup your entire tailnet via the admin console of your account, and then the control servers will distribute all the specified configuration to all your Tailscale agents.  That way the agents can work independently from the Control servers, and create peer-to-peer connections to the other agents in your tailnet.

The following diagram explains how you can control the devices in your tailnet via the Tailscale control servers:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/972b8bf2-c860-444c-8961-5d75d7a5359c)
