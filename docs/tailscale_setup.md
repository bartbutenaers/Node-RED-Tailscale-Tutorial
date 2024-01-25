# Setup a tailnet

This tutorial explains shortly how to get started with Tailscale, and setup your first tailnet.  Much more detailed information can be found on the internet.

## Create a Tailscale account
You need to create a Tailscale account once:
1. Navigate to www.tailscale.com
2. Press the ‘Get Started’ button.
3. You need to identify yourself at Tailscale, based on one of the available identity providers:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/d69b5d6e-f469-48e1-bf60-d32e7d73b23b" width="300">

   Select a provider from this list, for which you have already an existing account.
5. You will be redirected to the webpage of the selected identity provider, where you need to log in (using the your credentials for that provider).  For example Github:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/8bcdd254-a80c-4bbf-ad33-cfd255373e5d" width="300">

6. Optionally: it is even more safe if you have enabled 2-factor authentication on your identity provider account, because then you will have to enter extra a verification code (which you need to generate using the Google Authenticator app on your smartphone).
7. When logged in at your identity provider, your provider will ask you whether it is ok that Tailscale wants to access some minimal info from your provider account:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/8bed9d0b-7cd1-4327-bbfc-1ce66410ab02" width="300">

9.	Once you have authorized tailscale, your identity provider account is linked to your new Tailscale account.

## Adding devices to your tailnet
Tailnet agents can be installed on various platforms.  Since a tailnet needs minimal two agents, lets install a couple ones:
1. Install a Tailnet client on an Android smartphone like this:
   1. Install the Tailscale app via the Play Store:

      <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/b5c01561-4302-4450-9e06-225b6892bab9" width="300">

   2. When opening the Tailscale app, you need to authorize it by logging into your identity provider again and authorize Tailscale. 
   3. This way Tailscale recognizes your account, and the Tailscale agent on your smartphone will be added automatically to your tailnet.
2.	Install the Tailnet client for example on a Windows portable (see instructions [here](https://tailscale.com/download/windows)) and again authorized it via your identity provider.
3.	And at last install the Tailnet client on your Raspberry Pi running Node-RED (see ‘manual’ instructions [here](https://tailscale.com/download/linux/rpi) or specific for the Bookworm version [here](https://pkgs.tailscale.com/stable/#raspbian-bookworm)).
  
    Remark: after you executed the command “sudo tailscale up”, an authorization url will be displayed in the console.  Paste that url in a browser on any device, and once it is authenticated, the text 'Success' will appear in your command line:

    <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/b66a4b4a-fcd7-44ec-87a9-38bfd31c8994" width="800">

## Tailscale control servers
Tailscale offers a worldwide cluster of control servers which have multiple tasks:
+ DHCP server to assign virtual IP addresses (100.x.y.z) to Tailscale agents in your tailnet.
+ Host the admin console, where you can manage your tailnet.  
   Note: The admin console is only accessible via your identity provider.
+ Store the public LetsEncrypt certificates for all Tailnet agents, and distribute those to the other agents in your tailnet.
+ And so on...

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/6d68c4b7-82d0-4747-9c2d-01bee18effd5)

1. Navigate to the control servers via https://login.tailscale.com/admin
2. Logon to your tailnet via your identity provider
3. In the next sections we will configure our tailnet.

## Configure DNS in your tailnet
It is adviced to specify a virtual hostname for every Tailnet agent in your tailnet, because working with DNS has some advantages:
+ Hostnames are easier to remember compared to IP addresses.
+ It is not possible to request LetsEncrypt certificates for IP addresses (as common name).

Activating hostnames for agents in our tailnet is rather simple:
1. In the tabsheet *“Machines”* your see that every agent has received a virtual IP address 100.x.y.z:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/153777f7-5822-4782-9ba6-8298828800ab" width="800">
 
2. Via the “…” button you can (via menu *“Edit Machine Name”*) enter a logical virtual hostname.
3. Enable MagicDNS (in the DNS tabsheet), in order to be able to use the machine names in the tailnet via DNS:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/28820a23-31c9-436d-835c-c0061e0dc595" width="500">

4.	In the DNS tabsheet you can also find the name of your tailnet:

    <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/84f09eea-4712-4565-b292-4e90b4fc9cb1" width="800">


From now on, your devices are known by their machine name within the telnet.  So you can from your smartphone browser access Node-RED by simply navigating to http://your_raspberry_machine_name.your_tailnet_name.ts.net:1880

## Enable https in your tailnet
The data is transferred encrypted between the Tailscale agents in your tailnet, i.e. Tailscale already encrypts all the data using the Wireguard protocol.  However it is still useful to activate https on top of that, because the data is only encrypted ***between*** the Tailscale agents.  As a result, the data that enters at the first agent will leave the second agent in the exact same state.   In other words, the tailnet will simply pass your data transparent (without reading the content):

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/fcd1d485-47ea-4bf9-9587-4351829f4947)

There are a few reasons why you should setup a https connection yourself through the encrypted tailnet:
+ When you have very confidential data, and you don’t trust that Tailscale does not read the data (before encrypting it via Wireguard).
+ When the receiver expects https (based on signed certificates), like for example the Google Action Console servers.

In the tabsheet *"DNS"* you can enable https:

<img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/07aa2e7d-546f-443a-9801-cf9cc51b3156" width="500">

When https is enabled, this only means that the Tailscale agents are informed that they are allowed to request a LetsEncrypt certificate.  You are responsible yourself to trigger that request, and to use the resulting certificates to setup an https connection!

The process of setting up an https connection via Tailscale works like this:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/317d052a-4935-44ff-b063-5d83c0849843)

1. The process starts by executing the command `tailnet cert mytailhost`.
   Note: since the LetsEncrypt certificates have a validity period of 3 months, we will create a cron job dat executes this command monthly.
2. The Tailscale agent will generate once a key pair, and send a CSR (for common name “mytailhost.mytailnet.ts.net”) to the LetsEncypt servers.  The response will be a small temporary challenge file.
3. The Tailscale agent will request the Tailscale control servers to make this challenge public available on the domain “mytailhost.mytailnet.ts.net” as a DNS TXT record.
4. LetsEncrypt will try to find the challenge on this domain.
5. When found, LetsEncrypt knows that the Tailscale client owns the domain.  So LetsEncrypt will send the certificate to the Tailscale client.
6. The Tailscale client will store the certificate in the folder /var/lib/tailscale/certs where our webserver can fetch it, to use it to setup new https connections.
   
Note that both files will have owner and group ‘root’, which means the webserver cannot simply read their content.
