# Node-RED security basics

I created this repository to share my insights on setting up security for a Node-RED system. I’ve gathered these tips and tricks through my personal experiences and research. However, it’s important to note that while I strive for accuracy, I’m not an expert in this field.

The information provided in this article is intended to be a helpful guide, but it’s not exhaustive or infallible. Security is a complex and ever-evolving field, and what works today might not be sufficient tomorrow. Therefore, while these suggestions can enhance your system’s security, they don’t guarantee complete protection against all potential threats.

It’s also worth noting that this documentation is public, and could be read by anyone, including potential hackers. This means that there’s no *“security by obscurity”* - the security measures outlined here are based on their inherent strength, not their secrecy.

***Please be aware that I cannot be held responsible in any way if your system is compromised. This holds true even if the information I’ve shared is incomplete or incorrect. Implementing security measures and maintaining the security of your system is a complex task that ultimately lies with you.***

Please use this information as a starting point and consider consulting with a cybersecurity professional to ensure your system is as secure as it can be.

## Introduction

There are a few reasons why we would like to connect our Node-RED system to the (dangerous) internet, for example:
+	We want to view our Node-RED dashboard for watching our camera’s.
+	We want to pass speech commands to our Google Home device, which will send them (via the Google Action servers) to the endpoint of the node-red-contrib-google-smarthome node.
+ And so on...

On this readme page you will find a short introduction of each separate item, and you can drill down to see more information about the topic.

## Security basics
If you want to learn a bit more about security basics, you can read these tutorials:
+ Basics of https
+ Basics of (LetsEncrypt) certificates

## Avoid port forwarding
The easiest way to access Node-RED from the internet, is by using port forwarding.  However port forwarding should really avoided, which means that all ports should closed on your modem/router!  See here for more detail.

## Use Tailscale to access Node-RED
To setup secure network connectivity for your Node-RED (flow editor and/or dashboard), you will need to use a networking service.  Some of the available services also offer a limited free version, which will be most of the time sufficient for a simple home automation system.  Some examples:
+ *ZeroTier*: a very nice and easy to setup service.  But it lacked at the moment some features that I needed (like LetsEncrypt certificates, tunnels, ...).
+ *Cloudflare (Zero Trust)*: a full blown service, offering a wide spectrum of the security features.  However it is rather complex, so it didn't fit in my use case.  Because my familly should be able to maintain it, if I ever won't be around anymore.
+  And so on...

I decided to use *Tailscale*.  Because it offers all the security features that I need, and is 'quite' understandable.  

Tailscale allows you to create a virtual private network (VPN) between all your devices that have a Tailscale client installed.  In contradiction to a normal VPN, it is a peer-to-peer mesh network (called tailnet).  Which means that the data (normally) is communicated directly (encrypted) between your devices, instead of via a Tailscale server:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/580d9544-ee09-431a-bd41-8c1d80707a80)

This repository contains some tutorial with Node-RED specific Tailscale information, to help you get started with this:
+ How to access Node-RED via Tailscale, without port forwarding (see here).
+ How to access Node-RED indirectly, via a Tailscale relay connection (see here).
+ How to setup your tailnet, by installing Tailscale agent on your devices (see here).
+ How to setup a tunnel (secured with https and LetsEncrypt certificates), for example to allow Google to send speech commands (see here).
