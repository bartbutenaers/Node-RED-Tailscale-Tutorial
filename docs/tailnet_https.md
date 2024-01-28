# Use https in your tailnet
The data between the Tailscale agents in your tailnet is already encrypted, using the Wireguard protocol.  So it is safe enough to get started without activating https!

However it is still useful to activate https, because the data is only encrypted ***between*** the Tailscale agents.  Data that you send into a Tailscale agent will be encrypted by that agent, and the data will be decrypted by the agent that receives that encrypted data.  As a result if you send unencrypted data into your tailnet, it will leave your tailnet unencrypted.  What goes in will come out..

So the tailnet will simply pass your data transparent (without reading or changing the content), whether it is http or https traffic:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/fcd1d485-47ea-4bf9-9587-4351829f4947)

There are a few reasons why you should setup a ***https*** connection through the encrypted tailnet:
+ When you have very confidential data, and you don’t trust that Tailscale does not read the data (before encrypting it via Wireguard).
+ When the receiver expects https (based on signed certificates), like for example the Google Action Console servers.

## Enable https
In the tabsheet *"DNS"* of your Tailscale admin console, you can enable https:

<img src="https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/07aa2e7d-546f-443a-9801-cf9cc51b3156" width="500">

When https is enabled, this *ONLY* means that the Tailscale agents are informed that they are allowed to request a LetsEncrypt certificate.  You are responsible yourself to trigger that request, and to use the resulting certificates to setup an https connection!

## Generate LetsEncrypt certificates
To setup https, a certificate will be required.  Self-signed certificates are nothing but trouble, but fortunately Tailscale offers LetsEncrypt certificates out of the box.

We will need a LetsEncrypt certificate for each device in our tailnet that needs to offer https connections.  In our case we want a https connection to Node-RED, which means we will need to generate a LetsEncrypt certificate on our Raspberry Pi where Node-RED is running.

The process of generating LetsEncrypt certificates on a device in your tailnet works like this:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/317d052a-4935-44ff-b063-5d83c0849843)

1. Execute the command `tailscale cert your-machine-name.your-tailnet-name.ts.net` to request a certificate from LetsEncrypt.

   This is what happens in detail, in case you are wondering how this works:
   1. The first time you will execute this command, the Tailscale agent will generate a key pair (i.e. private key and public key) for this device.
   2. The Tailscale agent will send a CSR (for common name “your-machine-name.your-tailnet-name.ts.net”) to the LetsEncypt servers.  The response will be a small temporary challenge file.
   3. The Tailscale agent will request the Tailscale control servers to make this challenge public available on the domain “your_machine_name.your_tailnet_name.ts.net” as a TXT record in their public DNS.
   4. LetsEncrypt will try a couple of minutes to fetch that challenge file from you domain (“your_machine_name.your_tailnet_name.ts.net”).
   5. When LetsEncrypt has been able to fetch the challenge file, they know that we are owner of that domain.  So LetsEncrypt will send the certificate to the Tailscale client.

2. The certificate will arrive within a minute...
3. The Tailscale agent will store the certificate in the folder /var/lib/tailscale/certs.
4. Since LetsEncrypt certificates have a validity period of 3 months, we will need to renew the certificate periodically.  For example add this line to the crontab, to request a new certificate on the first day of every month:
   ```
   0 0 1 * * tailscale cert your-machine-name.your-tailnet-name.ts.net
   ```

## Setup a https connection
A this point you have LetsEncrypt certificates generated, but they are ***NOT*** being used yet!
Now it is time to setup a https connection, based on this LetsEncrypt certificate.

In a next tutorial we will setup a Caddy reverse proxy to use the certificate to setup http connections.  But at this moment we are going to keep it simple, and let Node-RED use it to setup http connections.  Note that is not optimal because if a hacker gains access to Node-RED, he will also be able to see your private key!  Keep that in mind if you want to keep doing it that way...

1. Since both files (cert and key) will have owner and group ‘root’, Node-RED cannot read their content (since Node-RED does not run as root user).  So you need e.g. copy both files e.g. to your .node-red folder an adjust the owner an group of the files to the Node-RED user.
2. Adjust the Node-RED settings.js file to start using this new key pair:
   ```
   requireHttps: true,
   https: function() {
      return {
         key: require("fs").readFileSync('/path/to/some/folder/your_machine_name.your_tailnet_name.ts.net.key'),
         cert: require("fs").readFileSync('/path/to/some/folder/your_machine_name.your_tailnet_name.ts.net.crt')
      }
   },
   ```
3. Restart Node-RED to activate the updated https settings.

Now test this setup, by navigating to your flow editor https://your-machine-name.your-tailnet-name.ts.net:1880/http-admin-root-path-if-available.
If the certifcate has been used, your browser should let you know that the ***connection is secure***.
