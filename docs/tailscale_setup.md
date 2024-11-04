# Setup a tailnet

This tutorial explains shortly how to get started with Tailscale, and setup your first tailnet.  Much more detailed information can be found on the internet.

## Create a Tailscale account
You need to create a Tailscale account once:
1. Navigate to www.tailscale.com
2. Press the ‘Get Started’ button.
3. You need to identify yourself at Tailscale, based on one of the available identity providers:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/d69b5d6e-f469-48e1-bf60-d32e7d73b23b" width="300">

   Remarks:
   + Select any provider from this list, for which you have an account already.  For example if you have a Google account (GMail etc), select Google. Google with then ask you if want to do this, and they will then verify your ID with Tailscale.
   + All the providers in the list will be able to provide Tailscale with information about your identitiy.  Because in one of the steps below you will tell your identitiy provider that they are allowed to share some identity information about you to Tailscale (e.g. your email address).
   + Since your Tailscale account will be linked to your provider account, always use the same provider when loggin in afterwards (in case you have existing accounts with these providers).  Otherwise you will get an error that there is ***no*** Tailscale account linked to your other provider account.
   + The identity provider gets ***no*** access to your tailnet, to your data or anything else.  The other way around, Tailscale will get only access to very ***limited*** data from your identity provider.  In step 7 you will see which data Tailscale wants to read from your identity provider (typical your email address, profile picture...), which you will then have to accept or reject.

5. You will be redirected to the webpage of the selected identity provider, where you need to log in (using the your credentials for that provider).  For example Github:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/8bcdd254-a80c-4bbf-ad33-cfd255373e5d" width="300">

6. Optionally: it is even more safe if you have enabled 2-factor authentication on your identity provider account, because then you will have to enter extra a verification code (which you need to generate using the Google Authenticator app on your smartphone) when logging into your security provider.
7. When logged in at your identity provider, your provider will ask you whether it is ok that Tailscale wants to access your provider account (to read some minimal info like e.g. your email address):

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/8bed9d0b-7cd1-4327-bbfc-1ce66410ab02" width="300">

9.	Once you have authorized tailscale (to access your provider account), your identity provider account is linked to your new Tailscale account.

## Adding devices to your tailnet
Tailscale agents can be installed on various platforms.  Since a tailnet needs minimal two agents, lets install 3 agents on different platforms:
1. Install a first Tailscale agent on an ***Android smartphone*** like this:
   1. Install the Tailscale app via the Play Store:

      <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/b5c01561-4302-4450-9e06-225b6892bab9" width="300">

   2. When opening the Tailscale app for the first time, you need to authorize it (to your tailnet) by logging into your identity provider again and authorize Tailscale. 
   3. Based on your identity provider account, the agent app knows to which Tailscale account it belongs.  That way, the Tailscale agent on your smartphone will be added automatically to your tailnet.

2.	Install a second Tailscale agent for example on a ***Windows 10 portable*** (see instructions [here](https://tailscale.com/download/windows)) and again authorized it via your identity provider.

3.	And at last install a third Tailscale agent on your Raspberry Pi running Node-RED (see ‘manual’ instructions [here](https://tailscale.com/download/linux/rpi) or specific for the Bookworm version [here](https://pkgs.tailscale.com/stable/#raspbian-bookworm)).  This agent will run as a systemd daemon named *"tailescaled"*.
  
    Remark: after you executed the command `sudo tailscale up`, an authorization url will be displayed in the console.  Paste that url in a browser on any device, and once it is authenticated in your tailnet, the text 'Success' will appear in your command line:

    <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/b66a4b4a-fcd7-44ec-87a9-38bfd31c8994" width="800">

## Show the tailnet devices
Now that you hace added some devices to your tailnet, you can have a first look at your virtual network:

1. Logon to the ***admin console*** on the Tailscale Control servers (via https://login.tailscale.com/start):
   
2. In the *"Machines"* tabsheet you can find a list of the devices in your tailnet:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/228aa24a-94c3-41c1-9e46-d5b82de2837d" width="800">

   Every device in the tailnet gets a fixed virtual IP address 100.x.y.z and a virtual hostname, both of which are only known within your tailnet. 

4. In the *"DNS"* tabsheet you can also find the name of your tailnet:

   <img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/ee30fb1d-2203-4234-b862-331b80cf05df" width="800">

   You can change this name, but you are limited to select a name from a list of automatically generated tailnet names.

## Approve devices
When a device is registered, it will join your tailnet automatically.  But you can secure your tailnet even more, by requiring every new device to be approved manually:

1. Login to the admin console
2. Activate once (in the *"Settings"* tabsheet) the manual device approval procedure:

   ![image](https://github.com/user-attachments/assets/1ea50a95-b406-4c16-a4ce-0755cb74dd1f)

3. Afterwards when a new device has requested to join your tailnet, you will see in the *"Machines"* tabsheet that you need to approve the device (before it can join your tailnet):

   ![image](https://github.com/user-attachments/assets/90b64190-95a3-46c8-b230-4ff74ef3e31b)

4. Click on the `...` button for the new device, and click on the *"Approve"* menu item.
5. Now the new device will join your tailnet.

## Remove devices
When a device from your tailnet is lost or stolen, it is advised to remove it immediately from you tailnet.  Because once a malicious user gets access to the device, he can access all devices on your tailnet.  You can remove the device from the tailnet, via the context menu when clicking on the *"..."* button:

![image](https://github.com/user-attachments/assets/ca701485-411e-4b4f-af4f-9ddbaada4721)
