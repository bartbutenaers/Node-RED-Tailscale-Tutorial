# Secure Node-RED via Tailscale

I created this repository to share my insights on setting up security for a Node-RED system.  This repository contains articles about setting up Tailscale to secure access to Node-RED.

After a lot of Node-RED systems had been hacked in 2023-2024, I decided to improve my own Node-RED security setup.  I especially wanted to get rid of my old port-forwarding setup, since most of the hacked Node-RED systems had a similar setup like mine.  You can read [here](https://github.com/bartbutenaers/Node-RED-security-basics/blob/main/docs/port_forwarding.md) why ***port-forwarding should be avoided!***

## The big bad internet
Obviously when you use Node-RED offline, you can minimize the risk of malicious users accessing your system.  However there are a number of nice use cases, that will only work by connecting Node-RED to the internet:
+	View the Node-RED dashboard remotely, to keep an eye on your house while not being at home.
+	Pass speech commands to a Google Home device, which will send them (via the Google Action servers) to the endpoint of the node-red-contrib-google-smarthome node.
+ And so on...

## Security basics
The following articles give some more technical details about some of the terminology used in this repository.  It is ***NOT*** required to read this, but it will give you some background information about how stuff works behind the scenes.  

+ ***Basics of https***: explains why you need a private key and public key for https (see [here](https://github.com/bartbutenaers/Node-RED-security-basics/blob/main/docs/https_introduction.md)).
+ ***Basics of (LetsEncrypt) certificates***: explains why you need LetsEncrypt certificates (see [here](https://github.com/bartbutenaers/Node-RED-security-basics/blob/main/docs/certificate_introduction.md)).

## Why using Tailscale
To setup a secure network connection to your Node-RED system (flow editor, dashboard and local endpoints) from the internet, it is higly advised to use a trusted ***networking service***.  Such a service is used between Node-RED and the internet, to handle all the complex security stuff.  You could install your own software locally to achieve a similar result, but then you will be responsible to setup and maintain it (e.g. install security patches as soon as a vulnerability is being reported).  That would become a tough nut to crack.  Let those security experts do their job, while you have have time to do fun stuff with Node-RED meanwhile ;-)

Some of those available services also offer a limited free version, which will be most of the time sufficient for a simple home automation system.  The following table shows some well-known services with a limited free version:

| Network service  | Advantages | Disadvantages |
| ------------- | ------------- | ------------- |
| ZeroTier  | Easy to setup  | No Letsencrypt certificates, no tunnels, ...  |
| Cloudflare (Zero Trust)  | Lots of features  | Pretty complex to setup  |
| Ngrok  | Very easy to setup  | No Letsencrypt certificates, limited traffic, ...  |

These services are very decent and popular choices, but they simply didn't match my personal use case:
+ My free time is too limited to setup and maintain something complex like Cloudflare.
+ One day I will need to explain my wife and kids how stuff works in our house, in case I ever won't be around anymore.  Which will be FAR from easy, so I 'try' to keep it as simple as possible...
+ I don't want to make my setup too complex, by having to setup my own reverse proxy (like e.g. Caddy, Nginx, ...).  I want my networking service to take care of that too.
+ I don't want any security related stuff inside Node-RED anymore, because the powers of Node-RED can be abused by hackers to disable its own security (once he would have arrived inside Node-RED).  For example I don't want to use my own [node-red-contrib-letsencrypt](https://github.com/bartbutenaers/node-red-contrib-letsencrypt) node anymore!  The network service should take care of the Letsencrypt certifiates too.
+ I don't want to setup my own client access control system (e.g. ip address whitelist, ...) to stop unauthorized devices from accessing my Node-RED system.  The network service should take care of such access control too.

Tailscale offers all the security features that I need, and it is 'quite' understandable.  Hopefully my tutorial gives you a general idea of how Tailscale can be used to achieve the above requirements.

## Introduction to Tailscale
Tailscale allows you to create a ***virtual private network (VPN)*** between all your devices, on which you have a Tailscale ***agent*** installed.  In contradiction to a normal VPN, Tailscale offers a ***peer-to-peer mesh network (called tailnet)***.  In other words instead of transferring all the data through a central server cluster (like most VPN services do), the data is communicated directly between devices in your tailnet.  Such direct connections between your devices offer multiple advantages:
+ The data can be exchanged very fast via such connections.
+ There are no traffic volume limitations, which is useful when transferring lots of data (e.g. video footage from IP cameras).
+ The tailnet will even continue working, even when the Tailscale servers would become temporarily unaccessible (unless you use their relay service).

When you want to access your Node-RED dashboard (running on a Raspberry Pi) from your smartphone, you need to install a Tailscale agent both on your Raspberry Pi and your smartphone.  Once you have added both devices (i.e. Tailscale agents) in your Tailscale account, they become part of your own ***tailnet***.  As a result both devices can communicate directly to each other via their Tailscale agents.  This means you can navigate with your smartphone browser to the (virtual!) hostname of your Raspberry Pi, to see your Node-RED dashboard:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/580d9544-ee09-431a-bd41-8c1d80707a80)

## Tailscale for Node-RED
This tutorial describes in detail how to get started with Tailscale.  The information is splitted in separate pages, to keep this readme compact and readable.

1. First we try to get an ***overview*** of our setup, to get some insights in how Tailscale can access Node-RED even when no port forwarding is setup on our modem/router: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_node_red.md).
2. Optionally you might read about the Tailscale ***relay*** servers, which will be useful in case of bad connections: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_relay.md).
3. Optionally you might read about the Tailscale ***control*** servers, which distribute your tailnet information from your account to all your Tailscale agents: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_control.md).
4. Setup your own ***tailnet***, by creating a Tailscale account and install some agents on your devices (which are added to your tailnet): see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_setup.md).
5. Once you have your own tailnet, it is useful to give each device a (virtual) logical hostname within your mesh network (for easy access).  So DNS needs to be setup: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_dns.md).
6. Make sure all our traffic to become ***https*** based on LetsEncrypt certificates: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailnet_https.md).
7. Finally setup a public tunnel (called ***funnel***) so third party services can access Node-RED, for example for Google Assistant: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_funnel.md).

## Disclaimer
It is important to note that while I try to strive for accuracy, I’m not an expert in this field.  The information provided in this repository is intended to be a helpful guide, but it’s not exhaustive or infallible. While this information can enhance your system’s security, there is no guarantee for complete protection against all potential threats.

It’s also worth noting that this documentation is public, and could be read by anyone, including potential hackers. This means that there’s no *“security by obscurity”* - the security measures outlined here are based on their inherent strength, not their secrecy.

***Please be aware that I cannot be held responsible in any way if your system is compromised. This holds true even if the information I’ve shared is incomplete or incorrect. Implementing security measures and maintaining the security of your system is a complex task that ultimately lies with you.***
