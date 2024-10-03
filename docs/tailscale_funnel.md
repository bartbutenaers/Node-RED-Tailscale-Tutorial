# Tailscale funnel

You need for example to make port 3001 of the node-red-contrib-google-smarthome node public available on the internet, to make sure the Google servers can connect to that port for sending voice comands.  But since we have previously deactivated all our port forwardings, you won't be able to access that port anymore remotely.

However such a private service can be made public accessible via the internet, by creating a secure tunnel to that service.  A Tailscale funnel is a secure tunnel from such a service, to a public endpoint in the cluster of Tailscale funnel servers.  Once setup, the internal service is exposed as a public service on the internet.  That way the requests from the public endpoint pass via the secure tunnel through your tailnet, but without access to any other device within your tailnet.  Indeed the requests can only access the local service that you have made public:

![image](https://github.com/user-attachments/assets/91d51c8d-568b-4d24-9bc6-dca9f65e062d)

***CAUTION!!!*** 
+ Make sure you have a https connection, based on LetsEncrypt certificates (via Tailscale)!  Because once your data leaves the encrypted funnel via the public endpoint, your data will be transported over the internet where hackers can intercept and read it.  You can achieve that by the `--https=443` in the command below.
+ Make sure that your local service access has secure access, before you make it public available through a tunnel!  Because once it becomes public, bots will detect it and try to hack it.  For example the node-red-contrib-google-smarthome node uses OAuth2 to secure access to it.

## Setup a funnel
Such a funnel can be setup like this:

1. Logon to the raspberry pi where Node-RED is running.
2. Start the funnel as a background process, so it keeps running after you have closed your terminal session:
   `sudo tailscale funnel --https=443 --bg --set-path /my_funnel http://localhost:3001`
3. The output of this command will show the public url (https://your_machine_name.your_tailnet_name.ts.net)  where you can access this funnel.
4. Open the url https://your_machine_name.your_tailnet_name.ts.net/check on your browser, and you should get the test page of the smarthome node:

   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/e69f56a3-85cb-4a4b-a17f-635b6b618a79)

   Remarks:
   - This will only work after you have completed the extensive configuration of your Google environment...
   - Make sure to test this from a browser on a device that is not part of your tailnet, or turn your Tailscale agent off on your device.  Otherwise you are not sure whether you are accessing the local service via your tailnet.

6. Optional.  You can stop the funnel via the command `sudo tailscale funnel --https=443 off`.  Use this command once you don't need a funnel anymore, to reduce the risk of getting hacked.
7. As long as the funnel is active, you will see it in the *"Machines"* tabsheet:
 
   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/e49f1111-3ecd-41c9-a670-1e96e72a90d7)

## Troubleshooting
Some tips to help you troubleshooting a funnel that is not working as expected:
+ Look in the receiver logs.  In this case the Node-RED logs where node-red-contrib-google-smarthome might have logged something.
+ Look in the Tailscale agent logs via `journalctl -u tailscaled`, to see if the requests have been blocked for some reason.
