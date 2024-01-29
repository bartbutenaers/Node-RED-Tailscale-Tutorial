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

## Show the tailnet devices
Now that you hace added some devices to your tailnet, you can have a first look at your virtual network:

1. Logon to the ***admin console*** on the Tailscale Control servers:
   
2. In the *"Machines"* tabsheet you can find a list of the devices in your tailnet:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/228aa24a-94c3-41c1-9e46-d5b82de2837d" width="800">

   Every device has been assigned a virtual hostname and a fixed virtual ip address 100.x.y.z

4. In the *"DNS"* tabsheet you can also find the name of your tailnet:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/ee30fb1d-2203-4234-b862-331b80cf05df" width="800">

   You can change this name, but you are limited to select a name from a list of automatically generated tailnet names.

## Access a virtual device
A first test is to check whether you can access (on your smartphone browser) your Node-RED dashboard:

1. Enter the virtual IP address of your Raspberry Pi in the browser on your smartphone:

   `http://your-device-virtual-ip-address:1880/ui`

2. If everything went well, the Tailscale agent on your smartphone should forward the browser http request to the Tailscale agent on your Raspberry Pi:

   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/7fdbf983-e8cd-4498-8d2a-09d7521fd22c)

3. In the next tutorials we will learn how to use a DNS name instead of an IP address, and how to setup https instead of http.
