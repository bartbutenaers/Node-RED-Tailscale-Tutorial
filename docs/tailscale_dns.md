# Configure DNS in your tailnet
When you have added devices to your tailnet, each device will get a virtual IP address 10.x.y.z from the DHCP server in the Tailscale control servers.  However it is adviced to specify a virtual host name for every Tailnet agent in your tailnet, because using host names has some advantages:
+ Host names are easier to remember compared to IP addresses.
+ It is not possible to request LetsEncrypt certificates for IP addresses (as common name).

Fortunately the Tailscale Control servers also offer a Magic DNS service.  A DNS (Domain Name System) can convert host names to IP addresses, which allows us to use virtual host names in our tailnet.

## Activating DNS
Activating hostnames for agents in our tailnet is rather simple:
1. In the tabsheet *“Machines”* your see that every agent has received via DHCP a virtual IP address 100.x.y.z:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/153777f7-5822-4782-9ba6-8298828800ab" width="800">
 
2. Via the “…” button you can (via menu *“Edit Machine Name”*) enter a logical virtual hostname.
3. Enable MagicDNS (in the DNS tabsheet), to activate the machine names as DNS names in your tailnet:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/28820a23-31c9-436d-835c-c0061e0dc595" width="500">

From now on, your devices are known by their machine name within the tailnet.

You can test this by navigating in your browser to the virtual host name of your Raspberry Pi, in order to open the Node-RED flow editor:
http://your_device_name.your_tailnet_name.ts.net:1880/your-http-admin-root-path-if-available
