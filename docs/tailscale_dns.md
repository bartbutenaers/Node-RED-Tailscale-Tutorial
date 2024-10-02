# Configure DNS in your tailnet
When you have added devices to your tailnet, each device will get a virtual IP address 10.x.y.z from the DHCP server in the Tailscale control servers.  

However it is adviced to activate DNS, because adressing devices using host names has some advantages:
+ Host names are easier to remember compared to IP addresses.
+ It is not possible to setup secure https, because LetsEncrypt certificates cannot be requested for IP addresses (as common name).

Fortunately the Tailscale Control servers offer a Magic DNS service.  Such a DNS (Domain Name System) can map host names to IP addresses, which allows us to use virtual host names in our tailnet (which will be resolved automatically to ip addresses behind the scenes).

## Activating DNS
Activating hostnames for agents in our tailnet is rather simple:
1. In the tabsheet *“Machines”* your see that every agent has received a virtual IP address 100.x.y.z via DHCP :

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/153777f7-5822-4782-9ba6-8298828800ab" width="800">

   Note that all devices get a default virtual hostname, which is a unique name generated from the device's OS hostname.
 
2. Via the “…” button you can (via menu *“Edit Machine Name”*) rename the hostname by a meaningful name.

3. Enable MagicDNS once (in the DNS tabsheet), to activate those virtual machine names as DNS names in your tailnet:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/28820a23-31c9-436d-835c-c0061e0dc595" width="500">

   From now on, your devices are known by their machine name within the tailnet.

4. You can test this by navigating in your browser to the virtual host name of your Raspberry Pi, in order to open the Node-RED dashboard D2 (which should now be available at http://your_device_name.your_tailnet_name.ts.net:1880/your-http-admin-root-path-if-available/dashboard):

   ![image](https://github.com/user-attachments/assets/14f6e307-38ac-4dee-b1e8-f366609f725a)
