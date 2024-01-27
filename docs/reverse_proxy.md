# Caddy as reverse proxy

Node-RED uses an ***ExpressJs webserver*** for its web application, which runs on top of a NodeJs javascript application server.  The Node-RED web application (inclusive the logon screen) will be served by that webserver:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/d9a536ef-f08a-4742-ac82-b49cc768edb7)

This means that a hacker will arrive on your webserver, where he will get to see your Node-RED logon screen.  Of course he needs to guess your username and password.  But be aware that as this moment he already arrived very close to your Node-RED system on your Raspberry Pi!

As soon as a hacker has access to the Node-RED logon page, he can exploit the ***security vulnerabilities*** of your NodeJs/ExpressJs/Node-RED system.  All these 3 levels work great, but their main target is not security!  Moreover all these levels depend heavily on third-party libraries as dependencies.  And these libraries again depend on other libraries, resulting in a large dependency tree.  If one of these dependencies contains a security vulnerability, then hackers will exploit it to hack your system.
Like the following cartoon explains very clearly:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/ec1a5528-932c-4a3a-b613-67838ebd9069)

So it is a bad idea to access your Node-RED system directly!  

## Reverse proxy
To solve this problem we will add an ***extra webserver*** in between, that acts as a reverse proxy.  A reverse proxy will forward requests to your Node-RED webserver:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/8607957d-1594-4929-a5d1-27bcefbc8ad8)

Webservers like Nginx, Caddy, … have security as a main target, and they will fix vulnerabilities very quickly.  That is why it is much safer to put those in between the bad internet and your Node-RED/ExpressJs/NodeJs stack.  Which means of course that you will need to ***update it frequently*** to the last stable release version!

Although Nginx is a very popular webserver, we will use Caddy instead:
+ Since Caddy 2.5 it supports Tailscale certificates out of the box: when a request arrives for *.ts.net, Caddy will automatically recognize it and use the certificates.
+ Caddy runs in a sandboxed environment for added security.

Some remarks about this drawing:
+ It is often advised to run Caddy on a separate machine, to isolate it completely from Node-RED.  That will of course make your physical setup more complex...
+ The Caddy webserver will be executed on your Raspberry Pi as 'caddy' user.  So even if a hacker can hack his way into the Caddy webserver, he can only execute commands with the caddy user that has limited permissions.
+ When a https request arrives from a client for a Tailnet subdomain (*.ts.net), Caddy will automatically check whether the Tailscale client has generated a LetsEncrypt certificate.  It will use that tailnet.ts LetsEncrypt certificate to setup the SSL connection with the client.
+ Only the root user and the caddy user can access the private key.
+ Afterwards in this repository there will be a tutorial about adding iptables firewall rules on your Raspberry, to make sure that your cannot directly access Node-RED from your home network.  So that you always have to pass Caddy to access Node-RED.  But that is for later...
+ The connection between Caddy and Node-RED is ***plain http***, so no https.  Since the data send between Caddy and Node-RED will not leave your Raspberry Pi, there is no need anymore to encrypt it.  If you still want to to do that and your Node-RED uses self-signed certificates, you will need to tell Caddy explicit (in the below Caddy config file) that he needs to accept your self-signed certificate as valid.  And otherwise you need to make sure that https is disable in your Node-RED settings.js file, as shown below.

## Adjust Node-RED settings
You might have already setup some basic security in your Node-RED settings.js file, which we will now need to disable again.  Because Caddy will be responsible for the security stuff now, and we want to avoid that both Caddy and Node-RED will do duplicate or even conflicting security related stuff.

1. As explained above, Node-RED should not require https connections anymore.  So disable it in the Node-RED settings.js file:
   ```
   //https: {
   //  key: require("fs").readFileSync('privkey.pem'),
   //  cert: require("fs").readFileSync('cert.pem')
   //},
   //requireHttps: true,
   ```

2. When Caddy does handle the basic authentication (i.e. request a username and password), the basic authentication in the Node-RED should be turned off.  Otherwise both Caddy and Node-RED will show a logon screen, which means you would have to enter your credentials twice.  So disable it in the Node-RED settings.js file:
   ```
   //adminAuth: {
   //    type: "credentials",
   //    users: [{
   //        username: "admin",
   //        password: "$2a$08$zZWtXTja0fB1pzD4sHCMyOCMYz2Z6dNbM6tl8sJogENOMcxWV9DN.",
   //        permissions: "*"
   //    }]
   //},
   ```
   ***TODO:*** is there a way to have basic authentication for the admin API, but not for the flow editor UI?

3. Restart Node-RED, to make sure these changes become active.

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

To implement the setup described in the above diagram, the Caddy configuration should behave like this:
+ Listen to port 8443 for https requests.
   Note: it would be easier if we would use the standard https port 443.  Because you don't need to specify that port 443 in your url's.  Indeed when you use https, your browser will automatically add port 443 to your request as a default https port (if no port has been specified).  However the Tailscale agent process (running on the same Raspberry Pi) is already using port 443 to allow communication by the Tailscale Control servers!
+	Ignore all requests that don't arrive from the machines in our tailnet (to limit access), i.e. only from IP addresses 100.x.y.z
+	When the Node-RED dashboard is being accessed, you perhaps might not want to show a logon screen to request a username and password.  Because it is only accessible via devices part of your tailnet.  Having to enter the credentials over and over again in a home automation system is quite not-user-friendly. 
+	When the Node-RED flow editor is being accessed, Caddy needs to ask username and password before forwarding this request to Node-RED.  Of course your flow editor is only accessible from devices within your tailnet, but imho it is better to have this extra layer of security.   
   Note: it is better to show a logon screen in Caddy, instead of using the Node-RED logon screen.  Because as mentioned above, we want to keep malicious users as far away from our Node-RED/ExpressJs/NodeJs stack as possible...

Such a typical Caddy config could look like this:
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
+ In an initial test phase it makes troubleshooting easier by having clear `respond` error messages.  However once everything is up and running, you might make these message texts a bit more cryptic, to avoid giving hackers to much clues about what is going wrong...
+ The password hash can be calculated using the command `caddy hash-password`. 
+ You need to restart Caddy every time you have changed the above config file (via `sudo systemctl restart caddy`).

## Check your certificate
Let's now test end-to-end whether everything works fine.

1. Navigate in your browser (on a device where a Tailscale agent is running) to your Node-RED flow editor app:
   https://your-machine-name.your-tailnet-name.ts.net:your-node-red-port/your-http-admin-root-if-available
2. If everything is fine, your browser should show you that your connection is secure.
3. You can now look at your LetsEncrypt certificate.  At this moment I can show it in my Chrome browser in a few steps, as shown in the following screenshot.  But Google changes their menu structure often, and for other browsers it will be different:

   ![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/e9772288-9ddd-4168-9635-fa816ed9cdbd)

4. Your browser will determine that your connection is secure, by checking 3 things in your certificate:
   + The certificate is ***issued to*** the hostname where you have been navigating to (i.e. same hostname as in your url).  Because that means it is the certificate of that particular machine.
   + The ceritifcate is ***issued by*** a certified Certification Authority that the browser trusts, in this case LetsEncrypt.
   + The certificate is still valid, because the validity period (of 3 months for LetsEncrypt certificates) is still not exceeded.
