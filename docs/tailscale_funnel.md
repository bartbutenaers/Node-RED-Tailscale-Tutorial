# Tailscale funnel
When a service - that is listening on a port of a device in your tailnet - needs to become public accessible via the internet, a secure tunnel should be setup to that service.  A Tailscale funnel is a secure tunnel from such a service/port, to a public endpoint in the cluster of Tailscale funnel servers.  Once setup, the internal service is exposed as a public service on the internet.

Make sure you have a https connection, based on LetsEncrypt certificates (via Tailscale)!  Because once your data leaves the encrypted funnel through the public endpoint, your data will be transported over the internet where hackers can intercept and read it.

You need for example a funnel to port 3001 of the node-red-contrib-google-smarthome node, to makes sure the Google servers can connect to that port for sending voice comands.

## Setup a funnel
Such a funnel can be setup like this:

1. Logon to the raspberry pi where Node-RED is running.
2. Start the funnel as a background process, so it keeps running after you have closed your terminal session:
   `sudo tailscale funnel -bg 3001`
3. The output of this command will show the public url (https://your_machine_name.your_tailnet_name.ts.net)  where you can access this funnel.
4. Open the url https://your_machine_name.your_tailnet_name.ts.net/check on your browser, and you should get the test page of the smarthome node:

   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/e69f56a3-85cb-4a4b-a17f-635b6b618a79)

   Note: this will only work if you completed the extensive configuration of your Google environment...

5. Optional.  You can stop the funnel via the command `sudo tailscale funnel --https=443 off`.  If you don't need a funnel anymore, it is better to stop it to reduce the risk of getting hacked.
6. As long as the funnel is active, you will see that live in the *"Machines"* tabsheet:
 
   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/e49f1111-3ecd-41c9-a670-1e96e72a90d7)

## Troubleshooting
Some tips to help you troubleshooting a funnel that is not working as expected:
+ Look in the receiver logs.  In this case the Node-RED logs where node-red-contrib-google-smarthome might have logged something.
+ Look in the Tailscale agent logs via `journalctl -u tailscaled`
