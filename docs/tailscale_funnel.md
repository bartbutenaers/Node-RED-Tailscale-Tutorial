# Setup a funnel
A Tailscale funnel is a secure tunnel from a service (listening to a port) on a device, to and endpoint in the cluster of Tailscale funnel servers.  From there on it is exposed as a public service on the internet.

Make sure you have a https connection, based on LetsEncrypt certificates (via Tailscale)!  Because once your data leaves the encrypted funnel through your tailnet, it will be transported over the internet where hackers can intercept it.

A funnel to for example port 3001 of the node-red-contrib-google-smarthome node can be setup like this:
1. Logon to the raspberry pi where Node-RED is running.
2. Start the funnel as a background process, so it keeps running after you have closed your terminal session:
   `sudo tailscale funnel -bg 3001`
3. The output of this command will show the public url (https://your_machine_name.your_tailnet_name.ts.net)  where you can access this funnel.
4. Open the url https://your_machine_name.your_tailnet_name.ts.net/check on your browser, and I get the test page of the smarthome node:

   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/e69f56a3-85cb-4a4b-a17f-635b6b618a79)

5. Optional.  Via the command `sudo tailscale funnel --https=443 off` you can stop the funnel.
6. As long as the funnel is active, you will see that very clearly in the *"Machines"* tabsheet:
 
   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/e49f1111-3ecd-41c9-a670-1e96e72a90d7)

Troubleshooting when the funnel is not working:
+ Look in the receiver logs.  In this case the Node-RED logs where node-red-contrib-google-smarthome might have logged something.
+ Look in the Tailscale agent logs via `journalctl -u tailscaled`
