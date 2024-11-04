# Tailscale control servers
Tailscale offers a worldwide cluster of control servers which perform multiple tasks:
+ They offer a ***DHCP server*** to assign virtual IP addresses (100.x.y.z) to new Tailscale agents in your tailnet.
+ They host the ***admin console***, which is a web interface that you can use to manage your tailnet: your devices, users, access control list, ...  The admin console is accessible via https://login.tailscale.com/start  
   Note: Every time you want to access the admin console, you will need to authenticate yourself via your preferred identity provider (Google, Microsoft, Github, ...).
+ They offer a ***DNS server*** to assign virtual hostnames to Tailscale agents in your tailnet.
+ They store the public ***LetsEncrypt certificates*** for all Tailnet agents, and distribute those certificates to all the agents in your tailnet.
+ And so on...

After your tailnet has been setup via the admin console of your account, the Tailscale Control servers will distribute all the specified configuration to all your Tailscale agents.  That way the agents can work independently from the Control servers, and create peer-to-peer connections to the other agents in your tailnet.  That way your tailnet will keep running, even when the Tailscale servers would be down temporarily.

The following diagram explains how you can control the devices in your tailnet via the Tailscale control servers:

![image](https://github.com/user-attachments/assets/c8f21b3d-9b74-4411-a1a6-3e4bbfcca9bd)

## Headscale
Tailscale is an open-source project on [Github](https://github.com/tailscale/tailscale).  However their Control Server software is ***not*** open source, but you are free to use it.

The [HeadScale](https://headscale.net/) project offers an open-source alternative for the Tailscale Control server software.  The project is not owned by Tailscale, but the contribute to it.  

One of the advantages of HeadScale is that you can host it yourself in your local network, which means you don't need an identity provider to login to your account.  However HeadScale is out of scope for this tutorial.
