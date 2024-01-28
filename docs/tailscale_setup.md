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

Note: In the DNS tabsheet you can also find the name of your tailnet:

<img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/84f09eea-4712-4565-b292-4e90b4fc9cb1" width="800">
