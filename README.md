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
You can skip these tutorials, if you are happy with the fact that you need a private key and a corresponding LetsEncrypt certificate to setup https.

But if you want to learn a bit more in depth about WHY you need these things and how that works, you can find detailed information in these tutorials:
+ ***Basics of https***, if you want to know why you need a private key and public key for https (see [here](https://github.com/bartbutenaers/Node-RED-security-basics/blob/main/docs/https_introduction.md)).
+ ***Basics of (LetsEncrypt) certificates***, if you want to know why you need certificates from LetsEncrypt (see [here](https://github.com/bartbutenaers/Node-RED-security-basics/blob/main/docs/certificate_introduction.md)).

## Avoid port forwarding
The easiest way to access Node-RED from the internet, is by using port forwarding.  However port forwarding should really avoided, which means that all ports should closed on your modem/router!  See here for more detail.

If you want to know why port forwarding is a bad idea, you can read more about it [here](https://github.com/bartbutenaers/Node-RED-security-basics/blob/main/docs/port_forwarding.md).

## Use Tailscale to access Node-RED
Since we cannot use port forwarding to access our Node-RED system, we need another way to do that.

To setup secure network connectivity for your Node-RED system (flow editor, dashboard and local endpoints), it is advised to use a ***networking service***.  Some of the available services also offer a limited free version, which will be most of the time sufficient for a simple home automation system.  Some examples of such services:
+ *ZeroTier*: a very nice and easy to setup service.  But it lacked at the moment some features that I needed (like LetsEncrypt certificates, tunnels, ...).
+ *Cloudflare (Zero Trust)*: a full blown service, offering a wide spectrum of all security features you can think of.  However it is pretty complex. 
+ *Tailscale*: a service that lies somewhere between ZeroTier and Cloudflare, both in terms of the number of security features and in terms of complexity.
+  And so on...

Although Cloudflare is very popular, I decided not to use it for a couple of (personal) reasons:
+ My free time is too limited to setup and maintain something complex.
+ One day I will need to explain my wife and kids how stuff works in our house, in case I won't be around anymore.  Which won't be an easy one...

Therefore I decided to use *Tailscale*.  Because it offers all the security features that I need, and is 'quite' understandable.

Tailscale allows you to create a ***virtual private network (VPN)*** between all your devices, on which you  have a Tailscale ***agent*** installed.  In contradiction to a normal VPN, it is a ***peer-to-peer mesh network (called tailnet)***.  In other words instead of transferring all the data through a central server cluster (like most VPN services do), the data is communicated directly between devices in your tailnet.

Suppose you want to show your Node-RED dashboard on your smartphone.  You install a Tailscale agent both on your smartphone and your Raspberry Pi, where Node-RED is running.  Now both devices are part of your tailnet, which means they can communicate directly to each other via their Tailscale agents.  So if you navigate with your smartphone browser to the (virtual!) hostname of your Raspberry Pi, you will see your dashboard:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/580d9544-ee09-431a-bd41-8c1d80707a80)

In fact it works quite simple.  You can get started with this easily by installing some agents, as described here.
--> dus gewoon 2 agents opzetten en http in Node-RED afzetten.

But as soon as you want to have a bit more functionality, you will need to do a bit more of reading:

This repository contains some tutorial with Node-RED specific Tailscale information, to help you get started with this:
+ How to access Node-RED via Tailscale, without port forwarding (see here).
+ How to access Node-RED indirectly, via a Tailscale relay connection (see here).
+ How to setup your tailnet, by installing Tailscale agent on your devices (see here).
+ How to setup a tunnel (secured with https and LetsEncrypt certificates), for example to allow Google to send speech commands (see here).
