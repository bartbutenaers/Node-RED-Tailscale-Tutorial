# Comparison with other solutions

While searching for a solution to secure my home automation system, I came across some other well-known third-party services.  Some of thes services also offer a limited free version, which will be most of the time sufficient for a simple home automation system.  The following table shows some of those services with a limited free version:

| Network service  | Advantages | Disadvantages |
| ------------- | ------------- | ------------- |
| ZeroTier  | Easy to setup  | no certificate management included, no tunnels, ...  |
| Cloudflare (Zero Trust)  | Lots of features  | Pretty complex to setup  |
| Ngrok  | Very easy to setup  | no certificate management included, limited traffic, ...  |
| VPN  | Easy to setup  | no certificate management included, limited traffic, ...  |

Remark: while most of these services have *"no certificate management"* included, they can used shared certificates but you will be responsible yourself for requesting and renewing them (e.g. LetsEncrypt certificates).

Although these services are very decent and popular choices, they simply didn't match my *personal use case*:
+ Not enough free time to setup Cloudflare, although Cloudflare is better compared to Tailscale in many areas.
+ A setup with Cloudflare is too complex to explain to the familly how it works.
+ To avoid having to setup my own reverse proxy,  the networking service should offer a reverse proxy.
+ The security should be setup outside Node-RED, because the powers of Node-RED can be abused by hackers to disable its own security.  For example I don't want to use my own [node-red-contrib-letsencrypt](https://github.com/bartbutenaers/node-red-contrib-letsencrypt) node anymore!  The network service should offer Letsencrypt certificates.

Fortunately Tailscale is a service that offers all the security features that I need, which is why I started using it.
