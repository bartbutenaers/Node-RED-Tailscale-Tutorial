# Tailscale funnel

You need for example to make port 3001 of the node-red-contrib-google-smarthome node public available on the internet, to make sure the Google servers can connect to that port for sending voice comands.  But since we have previously deactivated all our port forwardings, you won't be able to access that port anymore remotely.

However such a private service can be made public accessible via the internet, by creating a secure tunnel to that service.  A Tailscale ***funnel*** is a secure tunnel from such a service, to a public endpoint in the cluster of Tailscale funnel servers.  Once setup, the internal service is exposed as a public service on the internet.  Now the requests from the public endpoint pass - via the secure funnel - through your tailnet, but without access to any other device within your tailnet.  Indeed the requests can only access the local service that you have made public:

![image](https://github.com/user-attachments/assets/38c5ca87-d4f6-4db7-ac10-8cd2c3c46634)

***CAUTION:*** See security section below before you setup a funnel!!

## Setup a funnel
Such a funnel can be setup like this:

1. Make sure the local service (which we will make available public soon) does not use https, but ***plain http***.  Like said before (when serving a local service within your tailnet), it is not very useful to use https on a single device between the Tailscale agent and the local service.

   For example the node-red-contrib-google-smarthome node can be configured to not setup https on its own:

   ![image](https://github.com/user-attachments/assets/eb5b40f6-0c48-456d-bdaf-4293891ef446)

2. Logon to your Tailscale ***account***.
3. Make sure the reverse proxies in your Tailscale agents are ***allowed*** to setup funnels.  To do that, make sure the following attribute is available in your *"Access Control"* tabsheet:
   ```
   "nodeAttrs": [
      {
        "target": ["autogroup:member"],
        "attr":   ["funnel"],
      },
   ], 
4. Logon to the raspberry pi where Node-RED is running.
5. ***Start*** the funnel as a background process, so it keeps running after you have closed your terminal session:
   ```
   sudo tailscale funnel --https=443 --bg --set-path / http://localhost:3001
   ```
   The Tailscale agent will ask the Tailscale relay servers to setup an endpoint with a TLS proxy.  All https requests arriving on that endpoint will be forwarded through the funnel to the Tailscale agent.

   The advantage of using the standard https port 443 is that browsers will add it automatically to any https url, when not specified explicit.  However when port 443 is already in use on your device by another application, you could also use ports 8443 or 10000.  Any other port numbers are currently ***not*** supported for funnels.  This limitation is in most use cases not a show stopper, because you can serve multiple local services on the same https port (by using sub paths).
6. The ***output*** of this command will show the public url (https://your_machine_name.your_tailnet_name.ts.net)  where you can access this funnel.
7. Open the ***url*** https://your_machine_name.your_tailnet_name.ts.net/check on your browser, and you should get the test page of the smarthome node:

   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/e69f56a3-85cb-4a4b-a17f-635b6b618a79)

   Make sure to test this from a browser on a device that is not part of your tailnet, or turn your Tailscale agent off on your device.  Otherwise you are not sure whether you are accessing the local service via your tailnet (instead of via the public endpoint on the internet).

8. Optional.  You can ***stop*** the funnel via the command `sudo tailscale funnel --https=443 off`.  Use this command once you don't need a funnel anymore, to reduce the risk of getting hacked.  Note that an extra parameter `-f` can be added to tail the file, when you want to see live updates of the logs.
9. As long as the funnel is ***active***, you will see it in the *"Machines"* tabsheet:
 
   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/e49f1111-3ecd-41c9-a670-1e96e72a90d7)

## Show funnel status
Afterwards it is always possible to get the current status of your funnel(s).  Use the following command on Linux:
```
tailscale funnel status
```
The output of the command will not only show the current active funnels:
```
# Funnel on:
#     - https://<your_virtual_host_name>.<your_tailnet_name>.ts.net

https://<your_virtual_host_name>.<your_tailnet_name>.ts.net:1800 (tailnet only)
|-- / proxy http://localhost:1880

https://<your_virtual_host_name>.<your_tailnet_name>.ts.net (Funnel on)
|-- / proxy http://localhost:3001
```
As you can see, the output does also show the local services which are available within your *tailnet only*.  Which are the local services published previously via the command `tailscale serve ...`.  So in fact the `status` shows the entire configuration of the Tailscale agent's reverse proxy.  Which is a bit confusing because `tailscale serve status` also shows both the local and public services.

## Security
A funnel is a weak point in your security.  Because it allows all clients from the internet to access your local service, i.e. client devices which are ***not*** members of your tailnet.

Therefore it is really required to have some extra security:
+ Make sure that you have setup ***secure login*** to your local service, before you make it public available through a tunnel!  A minimal secure access would be login via username and password credentials.  Because once it becomes public, bots will detect it and try to hack it.  For example the node-red-contrib-google-smarthome node uses OAuth2 to secure access to it.
+ Make sure you have a ***https*** connection, based on LetsEncrypt certificates (via Tailscale)!  Otherwise it won't be possible to setup a funnel.  The people from Tailscale have made this requirement, because - once your data leaves the encrypted funnel via the public endpoint - your data will be transported over the internet where hackers can intercept and read it.  You can achieve a https connection, via the `--https=443` parameter in the command below.
+ It would be good to block malicious clients in the ***endpoint*** on the Tailscale Relay Funnel servers, because that is the entry point of all external traffic:
   + Block well known bots
   + IP whitelisting
   + Rate limiting to prevent DDOS attacks: limit the number of messages that can be send to your local service.
   + ...

   Cloudflare tunnels allow such kind of security, however unfortunately Tailscale doesn't.  I have registered a [feature request](https://github.com/tailscale/tailscale/issues/13809) for filtering control options in the Access Control list, and their lead developer was positive about the suggestion.  But currently there is nothing available, so malicious clients can access your server (e.g. Raspberry Pi) and you need to block them over there.
+ Install an ***extra reverse proxy*** (e.g. Caddy, NGinx, ...) in between the Tailscale agent and your local service.  Of course then you have 2 reverse proxies in series, which will make the setup again a bit more complex.
+ Add some ***firewall rules*** e.g. in iptables on Raspberry e.g. for rate limiting to limit the effect of DDOS attacks:
   ```
   iptables -A INPUT -p tcp --dport 3001 -m connlimit --connlimit-above 10 -j REJECT
   iptables -A INPUT -p tcp --dport 3001 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT
   ```
   ***TODO: this is NOT tested yet and should be reviewed!!***
+ While it is better to add security before a local service, it might not harm to put some extra security ***inside*** the local service.  For example I registered a [feature request](https://github.com/mikejac/node-red-contrib-google-smarthome/discussions/596) to make sure the node-red-contrib-google-smarthome node only allows access to IP addresses from Google API servers.  I am a bit stuck with that, due to [this](https://github.com/silverwind/cidr-tools/issues/24) issue.
+ ...

Note that the Tailscale Funnel Relay servers pass the client ip-address via the `Tailscale-Ingress-Src` http header, in case it is needed on the target device.

## Troubleshooting
Some tips to help you troubleshooting a funnel that is not working as expected:
+ Look in the receiver logs.  In this case the Node-RED logs where node-red-contrib-google-smarthome might have logged something.
+ Look in the Tailscale agent logs via `journalctl -u tailscaled`, to see if the requests have been blocked for some reason.
