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

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/17416f24-e533-4f38-bce5-a0e4bd94b90b)

Webservers like Nginx, Caddy, … have security as a main target, and they will fix vulnerabilities very quickly.  Which means of course that you will need to update it frequently to the last stable release version!

Although Nginx is very popular, we will use Caddy instead:
+ Since Caddy 2.5 it supports Tailscale certificates out of the box: when a request arrives for *.ts.net, Caddy will automatically recognize it and use the certificates.
+ Only the root user and the caddy user can access the private key
+ Caddy runs in a sandboxed environment for added security.

The Caddy webserver will be executed on your Raspberry Pi as 'caddy' user.  So even if a hacker can hack his way into the Caddy webserver, he can only execute commands with the caddy user that has limited permissions.
