# Tailscale funnel

A funnel is a public tunnel that can be used to make a local service public available on the internet, which means outside your tailnet.  Requests that are arriving through the funnel, will ***not*** have access to other services within your tailnet.

For example port 3001 of the node-red-contrib-google-smarthome node needs to become public available on the internet, to make sure the Google servers can connect to that port for sending voice comands.  But since we have previously deactivated all port forwardings, you won't be able to access that port anymore remotely (since all ports on your modem/router/firewall are now closed).

Once a funnel has been setup, you will be able to access your local service via an url that contains a subdomain of the Tailscale `ts.net` root domain (`https://<your_virtual_host_name>.<your_tailnet_name>.ts.net`).  Such an url is free, unlike Cloudflare tunnels that require you to buy your own domain name (at a yearly cost).

## Setup a funnel
***CAUTION:*** See security section below BEFORE you setup a funnel!!

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
5. ***Start*** the funnel to the node-red-contrib-google-smarthome node:
   ```
   sudo tailscale funnel --https=443 --bg --set-path / http://localhost:3001
   ```
   Explanation:
   + Listen to https requests on port 443, and start https connection (so create and renew a LetsEncrypt certificate).
   + Run as a background (bg) job, to make sure it keeps running when the command shell window is closed.
   + The root path is e.g. `/`, but you can choose something else.
   + All requests will be forwarded to port 3001 on localhost, where the node-red-contrib-google-smarthome node is listening.
   
   The Tailscale agent will ask the Tailscale relay servers to setup an endpoint with a TLS proxy.  All https requests arriving on that endpoint will be forwarded through the funnel to the Tailscale agent.

   The advantage of using the standard https port 443 is that browsers will add it automatically to any https url, when not specified explicit in the address bar.  However when port 443 is already in use on your device by another application, you could also use ports 8443 or 10000.  Any other port numbers are currently ***not*** supported for funnels.  Which is no problem in most cases, because you can serve multiple local services on the same https port (by using sub paths via `--set-path`).
7. The ***output*** of this command will show the public url (https://your_machine_name.your_tailnet_name.ts.net), where you can access this funnel.  It might take up to 10 minutes before the url will be operational!
8. Open that ***url*** (with sub-path *"/check"* i.e. https://your_machine_name.your_tailnet_name.ts.net/check) in your browser, and you should get the test page of the smarthome node:

   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/e69f56a3-85cb-4a4b-a17f-635b6b618a79)

   Make sure to test this from a browser on a device that is not part of your tailnet, or turn your Tailscale agent off on your device.  Otherwise you are not sure whether you are accessing the local service via your tailnet (instead of via the public endpoint on the internet).

9. Optional.  You can ***stop*** the funnel via the command `sudo tailscale funnel --https=443 off`.  Use this command once you don't need a funnel anymore, to reduce the risk of getting hacked.  
10. As long as the funnel is ***active***, you will see it in the *"Machines"* tabsheet:
 
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
As you can see, the output does also show the local services which are available within your *tailnet only*, which are the local services published previously via the command `tailscale serve ...`.  So in fact the `status` shows the entire configuration of the Tailscale agent's reverse proxy.  Which is a bit confusing because `tailscale serve status` also shows both the local and public services.

## Security
A funnel is a weak point in your security.  Because it allows all clients from the internet to access your local service, i.e. client devices which are ***not*** members of your tailnet.  Which means that even hackers and bots can access it, and hack the local service.

Moreover since the funnels are accessible via a Tailscale `ts.net` subdomain, it is easier for bots and hackers to find it.  In case of Cloudflare tunnels you have to buy your onw domain name, which hides your tunnel more due to security by obscurity...

Therefore it is really required to add some extra security to your local service:
+ Make sure that you have setup ***secure login*** to your local service, before you make it public available through a tunnel!  A minimal secure access would be login via username and password credentials.  For example the node-red-contrib-google-smarthome node uses OAuth2 to secure access to it.
+ Make sure you have a ***https*** connection, based on a LetsEncrypt certificate (provided by the Tailscale agent)!  Otherwise it won't even be possible to setup a funnel.  The people from Tailscale have made this requirement, because - once your data leaves the encrypted funnel via the public endpoint - your data will be transported over the internet where hackers can intercept and read it.  You can achieve a https connection, via the `--https=443` parameter in the command above.
+ It would be good to block malicious clients in the ***endpoint*** on the Tailscale Relay Funnel servers, because that is the entry point of all external traffic:
   + Block well known bots
   + IP whitelisting
   + Rate limiting to prevent DDOS attacks: limit the number of messages that can be send to your local service.
   + ...

   Cloudflare tunnels allow such kind of security, however unfortunately ***Tailscale funnels don't***.  I have registered a [feature request](https://github.com/tailscale/tailscale/issues/13809) for filtering control options in the Access Control list, and their lead developer was positive about the suggestion.  But currently there is nothing available, so malicious clients can access your server (e.g. Raspberry Pi) and you need to block them over there.
+ Install an ***extra reverse proxy*** (e.g. Caddy, NGinx, ...) in between the Tailscale agent and your local service.  Of course then you have 2 reverse proxies in series, which will make the setup again a bit more complex.
+ Add some ***firewall rules*** e.g. in iptables on Raspberry e.g. for rate limiting to limit the effect of DDOS attacks:
   ```
   iptables -A INPUT -p tcp --dport 3001 -m connlimit --connlimit-above 10 -j REJECT
   iptables -A INPUT -p tcp --dport 3001 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT
   ```
   ***TODO: this is NOT tested yet and should be reviewed!!***
+ While it is better to add security before a local service, it might not harm to put some extra security ***inside*** the local service:
   + I registered a [feature request](https://github.com/mikejac/node-red-contrib-google-smarthome/discussions/596) to make sure the node-red-contrib-google-smarthome node only allows IP addresses from Google API servers. 
   + For a NodeJs application like Node-RED or node-red-contrib-google-smarthome, it could be useful to implement rate limiting via the express-rate-limit npm library.
+ ...

Note that the Tailscale Funnel Relay servers pass the client ip-address via the `Tailscale-Ingress-Src` http header, in case it is needed on the target device.

## Troubleshooting
Some tips to help you troubleshooting a funnel that is not working as expected:
+ Look in the receiver logs.  In this case the Node-RED logs where node-red-contrib-google-smarthome might have logged something.
+ Look in the Tailscale agent logs via `journalctl -u tailscaled`, to see if the requests have been blocked for some reason.  Note that an extra parameter `-f` can be added to tail the file, when you want to see live updates of the logs.

## How stuff works behind the scenes
A Tailscale ***funnel*** is a secure tunnel between a local service and a public endpoint in the cluster of Tailscale funnel servers.

![image](https://github.com/user-attachments/assets/dcc162b9-9f1f-44c4-853c-8ad77e22b3ee)

1. Execute the `tailscale funnel ...` ***command*** to setup a funnel.
2. The Tailscale agent will configure its ***reverse proxy***, to listen to the specified port (e.g. 443).
3. The tailscale agent sends a ***request*** to the Tailscale Funnel Relay servers, to create a funnel to our local service (via port 443).
4. The Tailscale funnel relay servers will create a public funnel:
   + Setup public ***DNS records***, which refer to your subdomain on the Funnel Relay servers.
   + Then ***DNS propagation*** occurs, i.e. updating these DNS records across the internet to make them recognized globally.  That can take up to 10 minutes!
   + Once this is completed, a TCP proxy will be setup to forward all the https requests (for your subdomain) to your Tailscale agent.
5. Enter the url (see the output of the command from step 1) as ***fulfillment url*** in your Google Action Console (see [setup instructions](https://github.com/mikejac/node-red-contrib-google-smarthome/blob/master/docs/setup_instructions.md#create-project-in-actions-console) of the node-red-contrib-google-smarthome node).  That way Google knows where to send the voice commands.
6. When you ask your Google Home device to execute a command, it will forward the ***voice command*** to the Google Action servers.
7. The ***Google Action servers*** will forward the voice command to Tailscale Funnel Relay servers, via the fulfillment url you have specified.
8. The TCP proxy on the Tailscale Funnel Relay servers will forward the ***https request*** - via the funnel - to your Tailscale agent.
9. The reverse proxy in the Tailscale agent will do SSL termination, and pass a ***http request*** to your local service (in this case the node-red-contrib-google-smarthome node listening to port 3001).
10. The node-red-contrib-google-smarthome node will make the voice command available in your Node-RED ***flow***, so the requested action can be executed.  E.g. turn a light on.
