# Secure Node-RED via Tailscale

I created this repository to share my insights on setting up security for a Node-RED system.  This repository contains articles about setting up Tailscale to secure access to Node-RED.

After a lot of Node-RED systems had been hacked in 2023-2024, I decided to improve my own Node-RED security setup.  I especially wanted to get rid of my old port-forwarding setup, since most of the hacked Node-RED systems had a similar setup like mine.  Hopefully my articles can help users to get started with Tailscale for Node-RED.

During this tutorial we will discuss how to secure access for a typical Node-RED home automation system:
+ Access the Node-RED dashboard when you are not home from an Android smartphone.
+ Allow Google Assistant to access Node-RED to be able to execute actions for voice commands.

## The big bad internet
Some people simply say that Node-RED should be used only offline, to avoid malicious users accessing it from the internet.  However in that case a lot a lot of use cases for Node-RED would get lost.  Here are a few reasons to connect our Node-RED system to the (dangerous) internet, for example:
+	View the Node-RED dashboard remotely to watch video footage from IP cameras, to keep an eye on your house will not being at home.
+	Pass speech commands to a Google Home device, which will send them (via the Google Action servers) to the endpoint of the node-red-contrib-google-smarthome node.
+ And so on...

## Security basics
The following pages give some more detail about some of the stuff being used in this repository.  This information is ***NOT*** required to get started quickly, but it will give you some background information about how stuff works behind the scenes.  

+ ***Basics of https***: explains why you need a private key and public key for https (see [here](https://github.com/bartbutenaers/Node-RED-security-basics/blob/main/docs/https_introduction.md)).
+ ***Basics of (LetsEncrypt) certificates***: explains why you need LetsEncrypt certificates (see [here](https://github.com/bartbutenaers/Node-RED-security-basics/blob/main/docs/certificate_introduction.md)).
+ ***Avoid port forwarding***: explains why port forwarding is a bad idea (see [here](https://github.com/bartbutenaers/Node-RED-security-basics/blob/main/docs/port_forwarding.md)).

## Use Tailscale to access Node-RED
Since we don't want to use port forwarding to access our Node-RED system, we need another way to achieve that.

To setup a secure network connection to your Node-RED system (flow editor, dashboard and local endpoints) from the internet, it is advised to use a trusted ***networking service*** in between.  Such third-party services take care of all the complex security stuff, and they make sure that security patches are being installed as soon as available.  You can never achieve a similar result on your own.  Let those security experts do their job, while you have have time to do some more fun stuff with Node-RED meanwhile ;-)

Some of those available services also offer a limited free version, which will be most of the time sufficient for a simple home automation system.  Some well known examples of such services:

| Network service  | Advantages | Disadvantages |
| ------------- | ------------- | ------------- |
| ZeroTier  | Easy to setup  | No Letsencrypt certificates, no tunnels, ...  |
| Cloudflare (Zero Trust)  | Easy to setup  | Pretty complex to setup  |
| Ngrok  | Very easy to setup  | No Letsencrypt certificates, limited traffic, ...  |

Not saying at all that these solutions are no good!  They are very popular choices, but they simply didn't match my personal use case:
+ My free time is too limited to setup and maintain something complex like Cloudflare.
+ One day I will need to explain my wife and kids how stuff works in our house, in case I ever won't be around anymore.  Which won't be an easy one, so I 'try' to keep it as simple as possible...
+ I don't want to make my setup complex, by having to setup my own reverse proxy (like e.g. Caddy, Nginx, ...).  I want my networking service to take care of that too.
+ I don't want any security related stuff inside Node-RED anymore, because Node-RED is powerfull so it would not be difficult for a hacker to disable such security (once he would have arrived inside Node-RED).  So for example I don't want to use my own [node-red-contrib-letsencrypt](https://github.com/bartbutenaers/node-red-contrib-letsencrypt) node anymore!  The network service should take care of the Letsencrypt certifiates too.
+ I don't want to setup my own filters (e.g. ip address whitelist, ...) to stop unauthorized devices from accessing my Node-RED system.  The network service should take care of such access control too.

After a while I decided to use *Tailscale*.  Because it offers all the security features that I need, and it is 'quite' understandable.  Although I had some hard time to figure out from their documentation how I had to configure it for my long list of requirements.  Hopefully my tutorial gives you a general idea of what is needed, and how it can be implemented.

Tailscale allows you to create a ***virtual private network (VPN)*** between all your devices, on which you have a Tailscale ***agent*** installed.  In contradiction to a normal VPN, it offers a ***peer-to-peer mesh network (called tailnet)***.  In other words instead of transferring all the data through a central server cluster (like most VPN services do), the data is communicated directly between devices in your tailnet.  Due to these direct connections between your devices offer multiple advantages:
+ The data connections are very fast.
+ There are no traffic volume limitations.
+ It will even work when their cloud servers would be unaccessible (unless you use their relay service).

Let's see how such a P2P mesh network looks like.  Suppose you want to access your Node-RED dashboard on your smartphone, with all ports closed on your modem/router.  You install a Tailscale agent both on your smartphone and your Raspberry Pi, where Node-RED is running.  After you add both devices in your Tailscale account, they become part of your own custom ***tailnet*** (which is they call a virtual private mesh network).  That means both devices can now communicate directly to each other via their Tailscale agents.  So if you navigate with your smartphone browser to the (virtual!) hostname of your Raspberry Pi, you will see your dashboard:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/580d9544-ee09-431a-bd41-8c1d80707a80)

## Tailscale for Node-RED
This tutorial describes how to get started with Tailscale.  The information is splitted in separate pages, to keep this readme compact and readable.

1. First we try to get an ***overview*** of our setup, to get some insights in how Tailscale can access Node-RED even when no port forwarding is setup on our modem/router: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_node_red.md).
2. Optionally you might read about the Tailscale ***relay*** servers, which will be useful in case of bad connections: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_relay.md).
3. Optionally you might read about the Tailscale ***control*** servers, which distribute your tailnet information from your account to all your Tailscale agents: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_control.md).
4. Setup your own ***tailnet***, by creating a Tailscale account and install some agents on your devices (which are added to your tailnet): see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_setup.md).
5. Once you have your own tailnet, it is useful to give each device a (virtual) logical hostname within your mesh network (for easy access).  So DNS needs to be setup: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_dns.md).
6. Make sure all our traffic to become ***https*** based on LetsEncrypt certificates: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailnet_https.md).
7. Finally setup a public tunnel (called ***funnel***) so third party services can access Node-RED, for example for Google Assistant: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_funnel.md).

## Disclaimer

I’ve gathered these tips and tricks through my personal experiences and research. However, it’s important to note that while I try to strive for accuracy, I’m not an expert in this field.  The information provided in this repository is intended to be a helpful guide, but it’s not exhaustive or infallible. Security is a complex and ever-evolving field, and what works today might not be sufficient tomorrow. Therefore, while these suggestions can enhance your system’s security, they don’t guarantee complete protection against all potential threats.

It’s also worth noting that this documentation is public, and could be read by anyone, including potential hackers. This means that there’s no *“security by obscurity”* - the security measures outlined here are based on their inherent strength, not their secrecy.

***Please be aware that I cannot be held responsible in any way if your system is compromised. This holds true even if the information I’ve shared is incomplete or incorrect. Implementing security measures and maintaining the security of your system is a complex task that ultimately lies with you.***
