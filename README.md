# Secure Node-RED via Tailscale

I created this repository to share my insights on setting up (free) security for a Node-RED system, more particular based on [Tailscale](https://tailscale.com/).  

After a lot of Node-RED systems had been hacked in 2023-2024, I decided to improve my own Node-RED security setup.  I especially wanted to get rid of my old port-forwarding setup, since most of the hacked Node-RED systems had a similar setup like mine.  We will discuss further on why ***port-forwarding should be avoided!***

This tutorial only teaches you the basics of Tailscale.  Afterwards I recommend to have a look at the Tailscale documentation (see [here](https://tailscale.com/kb/1017/install)) for more details.  They also offer some video tutorials on YouTube.

Please read the ***disclaimer*** at the bottom of this readme page, before you get started!

Thanks to [Julian Knight](https://github.com/TotallyInformation) who devoted a part of his life to help users to keep their Node-RED systems secure.  He teached me a lot about security, and was so kind to review this tutorial!  Have a look at his [blog](https://it.knightnet.org.uk/) which contains lots of useful articles.

## Issues and pull requests
***Pull-requests*** are very welcome, to help improving the information in this tutorial.   That is the only way we can help each other to improve our Node-RED security.  See the [contributing guide](https://github.com/bartbutenaers/Node-RED-Tailscale-Tutorial/blob/main/docs/contributing.md) how to make changes yourself.

***Issues*** can be used e.g. to mention if a part of the documentation is not correct or incomplete.  But when you have a Tailscale issue with your setup, post it directly in the Tailscale [repo](https://github.com/tailscale/tailscale/issues) or create a discussion in the Node-RED Discourse forum (where you can mention me to get my attention).  But ***DON'T***  post such issues in this repo, because I don't have the time to troubleshoot individual use cases!!!

## The big bad internet
Obviously when you would use Node-RED only offline, you could minimize the risk of malicious users accessing your system.  However there are a number of nice use cases, that will only work by connecting Node-RED to the internet:
+	View the Node-RED dashboard remotely, to keep an eye on your house while not being at home.
+	Pass speech commands to a Google Home device, which will send them (via the Google Action servers) to the endpoint of the node-red-contrib-google-smarthome node.
+ And so on...

## Security basics
The following articles provice technical details about some of the terminology used in this repository.  It is ***NOT*** required to read this, because it is rather heavy technical information.  But it will give you some background information about how stuff works behind the scenes.  

+ ***Basics of https***: explains why you need a private key and public key for https (see [here](https://github.com/bartbutenaers/Node-RED-Tailscale-Tutorial/blob/main/docs/https_introduction.md)).
+ ***Basics of (LetsEncrypt) certificates***: explains why you need to create a (LetsEncrypt) certificate from your public key (see [here](https://github.com/bartbutenaers/Node-RED-Tailscale-Tutorial/blob/main/docs/certificate_introduction.md)).

## Why using Tailscale
To setup a secure network connection to your Node-RED system (flow editor, dashboard, local endpoints, ...) from the internet, it is higly advised to use a trusted third-party ***networking service***.  Such a service is used between Node-RED and the internet, to handle all the complex security stuff:

![image](https://github.com/user-attachments/assets/8bfa3d46-787b-4c1b-a2b0-abe41a669398)

You could also prefer to install your own ***reverse-proxy*** software locally to achieve a similar result, but that has some disadvantages:
+ Setup a reverse-proxy (e.g. Caddy, Nginx, ...) is quite complex.
+ Maintaining your setup continiously is a must (e.g. install security patches as soon as vulnerabilities are being reported).
+ Setup of access control is a must, to reduce the number of devices that can access your system.
+ And so on ...

Such a reverse-proxy setup would become a tough nut to crack for a lot of Node-RED users.  Let the security experts from the third-party service do their job, while you have have time to do fun stuff with Node-RED meanwhile ;-)

I had a look at several third-party services, but at the end I decided that Tailscale was the best solution for my use case.  You can look [here](https://github.com/bartbutenaers/Node-RED-Tailscale-Tutorial/blob/main/docs/comparison.md) for a comparison of Tailscale with some other services.

## Introduction to Tailscale
Tailscale allows you to create a ***virtual private network (VPN)*** between all your devices, as long as you have a Tailscale ***agent*** installed on all those devices.  In contradiction to a normal VPN, Tailscale offers a ***peer-to-peer mesh network*** which is called a ***tailnet***.  In other words instead of transferring all the data through a central server cluster (like most VPN services do), the data is communicated directly between devices in your tailnet.  Such direct connections between your devices offer multiple advantages:
+ The data can be exchanged very fast via such connections.
+ There are no traffic volume limitations, which is useful when transferring lots of data (e.g. video footage from IP cameras).  Only for public funnels, there is a traffic limitation.
+ The tailnet will continue working, even when the Tailscale servers would become temporarily unaccessible.  Except if you use their relay or funnel service.

When you want to access your Node-RED dashboard (running on a Raspberry Pi) from your smartphone, you need to install a Tailscale agent both on your Raspberry Pi and your smartphone.  Once you have added both devices (i.e. Tailscale agents) in your Tailscale account, they become part of your own ***tailnet***.  As a result both devices can communicate directly to each other via their Tailscale agents.  This means you can navigate with your smartphone browser to the (virtual!) hostname of your Raspberry Pi, to see your Node-RED dashboard:

![image](https://github.com/bartbutenaers/Node-RED-Tailscale-Tutorial/assets/14224149/580d9544-ee09-431a-bd41-8c1d80707a80)

You can read [here](https://github.com/bartbutenaers/Node-RED-Tailscale-Tutorial/blob/main/docs/port_forwarding.md) why port forwarding should be avoided, and how Tailscale can work without port forwarding.

## Tailscale for Node-RED
This tutorial describes in detail how to get started with Tailscale.  The information is splitted in separate pages, to keep this readme compact and readable.

1. Optionally you might read about the Tailscale ***relay*** servers.  These servers will be used to connect two Tailscale agents, in case a direct connections cannot be setup: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_relay.md).
2. Optionally you might read about the Tailscale ***control*** servers, which distribute your tailnet information from your account to all your Tailscale agents: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_control.md).
3. Setup your own ***tailnet***, by creating a Tailscale account, and install Tailscale agents on your devices (which are added to your tailnet): see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_setup.md).
4. Specify a (virtual) logical hostname for each device within your tailnet (both for easy access and https), i.e. setup DNS in your tailnet: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_dns.md).
5. Configure Node-RED to allow it to be accessed from the tailnet, using plain http connections within your tailnet: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/edit/main/docs/tailnet_plain_http.md).
6. Optionally you might read about how ***reverse proxies*** work, to understand better the sub-path routing that we are going to use: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale-Tutorial/blob/main/docs/reverse_proxy.md).
7. Serve Node-RED als a local service via ***https***, based on LetsEncrypt certificates: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailnet_serve_https.md).
8. Setup a public tunnel (called ***funnel***) to allow third-party services to access Node-RED, for example for Google Assistant: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_funnel.md).
9. Make some other local services available within your tailnet, to make sure you can access them even when you are not at home: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale-Tutorial/tree/main/docs).
10. Finally specify which devices are allowed to communicate with each other (within your tailnet), using Access Control Lists (ACL): see [here](https://github.com/bartbutenaers/Node-RED-Tailscale/blob/main/docs/tailscale_access_control.md).

## Troubleshooting
There is a lot of stuff involved, so troubleshooting isn't always easy.  I have created a list of tips from my own experience: see [here](https://github.com/bartbutenaers/Node-RED-Tailscale-Tutorial/blob/main/docs/troubleshooting.md).

## !!! Disclaimer !!!
It is important to note that while I try to strive for accuracy, I’m not an expert in this field.  The information provided in this repository is intended to be a helpful guide, but it’s not exhaustive or infallible. While this information can enhance your system’s security, there is no guarantee for complete protection against all potential threats.

It’s also worth noting that this documentation is public, and could be read by anyone, including potential hackers. This means that there’s no *“security by obscurity”* - the security measures outlined here are based on their inherent strength, not their secrecy.

***Please be aware that I cannot be held responsible in any way if your system is compromised. This holds true even if the information I’ve shared is incomplete or incorrect. Implementing security measures and maintaining the security of your system is a complex task that ultimately lies with you.***
