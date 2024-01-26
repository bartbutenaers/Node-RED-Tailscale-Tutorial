# Caddy as reverse proxy

Node-RED uses an ExpressJs webserver for its web application, which runs on top of a NodeJs javascript application server.  The Node-RED web application (inclusive the logon screen) will be served by that webserver:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/d9a536ef-f08a-4742-ac82-b49cc768edb7)

This means that a hacker - via your logon screen - will arrive on your webserver, where he will be blocked due to incorrect credentials.  But he arrived very close to your Node-RED system on your Raspberry Pi already!

As soon as a hacker has access to the Node-RED logon page, he can exploit the security vulnerabilities of your NodeJs/ExpressJs/Node-RED system.  All these 3 levels work great, but their main target is not security!  Moreover all these levels depend heavily on third-party libraries as dependencies.  And these libraries again depend on other libraries, resulting in a large dependency tree.  If one of these dependencies contains a security vulnerability, then hackers will exploit it to hack your system.
Like the following cartoon explains very clearly:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/ec1a5528-932c-4a3a-b613-67838ebd9069)

So it is a bad idea to access your Node-RED system directly!  

## Reverse proxy
To solve this problem we will add an extra webserver in between, that acts as a reverse proxy (i.e. forward requests to your Node-RED webserver):

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/0773b89f-5002-4630-a87f-fb53c761ab4b)

Webservers like Nginx, Caddy, … have security as a main target, and they will fix vulnerabilities very quickly.  Which means of course that you will need to update it frequently to the last stable release version!

Although Nginx is very popular, we will use Caddy instead:
+ Since Caddy 2.5 it supports Tailscale certificates out of the box: when a request arrives for *.ts.net, Caddy will automatically recognize it and use the certificates.
+ Only the root user and the caddy user can access the private key
+ Caddy runs in a sandboxed environment for added security.

Some remarks about this drawing:
+ The Caddy webserver will be executed on your Raspberry Pi as 'caddy' user.  So even if a hacker can hack his way into the Caddy webserver, he can only execute commands with the caddy user that has limited permissions.
+ When a https request arrives for a Tailnet subdomain (*.ts.net), Caddy will automatically check whether the Tailscale client has generated a LetsEncrypt certificate.  It will use that certificate to setup the SSL connection with the client.

## Install Caddy on Raspberry Pi
1. Caddy can be installed on Raspberry using following commands:
   ```
   sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
   curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
   curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
   sudo apt update
   sudo apt install caddy
   ```
2. To allow the caddy user to access the certificate file (which requires root access), you need to add `TS_PERMIT_CERT_UID=caddy` to the /etc/default/tailscaled file (and restart the tailescaled daemon):

   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/ce13f156-4397-40df-93f3-ceea873ea331)

3. The first time you start Caddy on a server, it tries to generate a CA root certificate in the folder /usr/local/share/ca-certificates folder.  Although we don't need that certificate, you need to give Caddy once the permissions to write into that directory via `sudo caddy trust`.

4. Once installed you can check whether Caddy is running as a systemd background process via `systemctl status caddy`:

   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/e89de4ea-3e6a-4e3b-9c4b-f39f5b1308ab)

## Caddy configuration
The Caddy configuration is stored in /etc/caddy/Caddyfile.

To make sure Caddy can implement the forwarding described in the above diagram, the Caddy configuration should behave like this:
+ Listen to port 8443 for https requests.  The reason we don't use the default https port (443) is that this port is already being used by the Tailscale agent process on our Raspberry.
+	Ignore all requests that don't arrive from the machines in our tailnet (to limit access), i.e. only from IP addresses 100.x.y.z
+	When the Node-RED dashboard is being accessed, you perhaps might not ask a username and password.  To make it a bit more comfortable for e.g. using it on tablets inside the house. But of course you can ask a username and password also for your dashboard.  ***TODO:*** is this acceptable or not?
+	When the Node-RED flow editor is being accessed, Caddy needs to ask username and password before forwarding this request to Node-RED.  Because it is better to block users inside Caddy, instead of inside Node-RED.

When Caddy does handle the basic authentication (i.e. request a username and password), the basic authentication in the Node-RED settings.js file should be turned off:
```
/** To password protect the Node-RED editor and admin API, the following
 * property can be used. See https://nodered.org/docs/security.html for details.
 */
//adminAuth: {
//    type: "credentials",
//    users: [{
//        username: "admin",
//        password: "$2a$08$zZWtXTja0fB1pzD4sHCMyOCMYz2Z6dNbM6tl8sJogENOMcxWV9DN.",
//        permissions: "*"
//    }]
//},
```
Otherwise the Node-RED logon screen would appear after you passed the Caddy logon screen, resulting in having to enter your credentials twice.
***TODO:*** is there a way to have basic authentication for the admin API, but not for the flow editor UI?

Such a Caddy config could look like this:
```
(common_checks) {
    # Only allow access from a Tailscale private IP address range, which is by default from 100.64.0.0 to 100.127.255.255.
    # Use the CIDR (Classless Inter-Domain Routing) notation, by specifying a bit mask:
    # /10 means that the first 10 bits need to be the same as in the specified IP address, which means 100.64.0.xxx
    # In that case return status code 403, which means it is forbidden to access the specified resource.
    @notTailscale {
        not remote_ip 100.64.0.0/10
    }
    respond @notTailscale 403 "Only access from Tailscale nodes allowed"
}

{
    # Listen to a custom https port, because the tailescaled daemon alreayd is listening to the standard https port 443
    https_port 8443
}

# !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
# ENTER YOUR OWN TAILSCALE MACHINE NAME AND YOUR TAILNET NAME
# !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
# Requests for the new Node-RED (both http and https).
# Since a *.ts.net hostname is specified, Caddy will automatically get the LetsEncrypt certificates from the Tailscale client.
your-tailscale-machine-name.your-tailnet-name.ts.net {
    import common_checks

    # !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    # COMMENT THIS SECTION WHEN YOUR FLOW EDITOR IS AVAILABLE VIA THE ROOT PATH, I.E. WHEN YOU HAVE NOT SPECIFIED HTTPADMINROOT IN YOUR SETTINGS.JS FILE
    # !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    # When the Node-RED flow editor is accessed (at the base path), the username/password should be specified.
    # This is not required for the Node-RED dashboard, to improve user experience
    handle / {
        respond "The requested Node-RED site is not available"
    }

    # !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    # ENTER THE HTTPADMINROOT PATH FROM YOUR SETTINGS.JS FILE OR SIMPLY / IF YOU DON'T USE THAT SETTING
    # !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    # When the Node-RED flow editor is accessed (at the subpath '/your-http-admin-root-path' as specified in the settings.js file as httpAdminRoot),
    # the username/password should be specified.  This is not required for the Node-RED dashboard, to improve user experience.
    handle /your-http-admin-root-path {
        basicauth {
            # !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
            # ENTER YOUR USERNAME AND PASSWORD THAT YOU REQUIRE TO ACCESS YOUR NODE-RED FLOW EDITOR (AND PERHAPS DASHBOARD)
            # !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
            your-user-name your-password-hash
        }
    }

    # Reverse proxy to Node-RED
    reverse_proxy localhost:1880 {
        # !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
        # OPTIONALLY YOU CAN PASS SOME SECRET TOKEN TO NODE-RED, AS EXPLAINED FURTHER ON IN THIS REPOSITORY
        # !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
        # Add a http header so Node-RED can recognize requests from Caddy
        header_up X-Secret-Token "your-secret-token"
    }

    # Logging configuration for this site
    log {
        # Specify the output file for access logs
        output file /var/log/caddy/raspberrypi.access.log
        format console
    }
}

# !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
# ENTER YOUR OWN TAILSCALE MACHINE NAME AND YOUR TAILNET NAME
# !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
# Requests for the node-red-contrib-google-smarthome node endpoint
# Since a *.ts.net hostname is specified, Caddy will automatically get the LetsEncrypt certificates from the Tailscale client.
your-tailscale-machine-name.your-tailnet-name.ts.net:3000 {
    #import common_checks

    reverse_proxy http://localhost:3001
    
    # Logging configuration for Google voice assistant
    log {
        # Specify the output file for access logs
        output file /var/log/caddy/google-voice-assistant.access.log
        format console
    }
}

# Handle all other requests, that don't match any of the site blocks above
* {
    respond "The requested site is not available"

    # Logging configuration for all other requests
    log {
        # Specify the output file for access logs
        output file /var/log/caddy/general.access.log
        format console
    }
}
```
Some remarks:
+ We use 3 different log files to separate the logs a bit functionally.
+ In an initial test phase it makes life a bit easier to troubleshoot by having clear `respond` error messages.  However once everything is up and running, you might make this message texts a bit more cryptic, to avoid giving hackers to much clues about what is going wrong...
+ The password hash can be calculated using the command `caddy hash-password`. 
+ You need to restart Caddy every time you have changed the above config file (via `sudo systemctl restart caddy`).
