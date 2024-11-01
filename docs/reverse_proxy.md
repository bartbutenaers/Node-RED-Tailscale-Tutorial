# Reverse proxy

There are an immense amount of tutorials and videos available on the web about reverse proxies, so please have a look at these when you have problems understanding the concept.

A reverse proxy is a server/application/cloudservice that sits in front of web servers to intercept and inspect incoming client requests, before forwarding those requests to the web servers.  It can be used to:
+ Add some extra security to the web servers, for example by filtering requests from unauthorized clients.
+ Add load balancing, to distribute the client requests evenly across multiple web servers.
+ Forwarding requests via path-based routing, for example to make multiple applications accessible via a single port.
+ Handle incoming https connections, and do SSL/TLS termination (i.e. decrypt the traffic and forward the requests as plain http).
+ Manage certificates, for example automatically request and periodically renew LetsEncrypt certificates.
+ ...

Throughout this tutorial we will use the reverse proxy in the Tailscale agent, which we will use to handle https connections (via LetsEncrypt certificates) and path-based routing.

## Path-based routing

Path-based routing means that the reverse proxy looks at the URL inside the incoming http requests, to decide to which application (containing a web server) those requests need to be ***forwarded***.

Let's take a look at the path-based routing scheme that will be constructed step-by-step, throughout the remaining chapters of this tutorial:

![image](https://github.com/user-attachments/assets/74ad90e5-40a9-452b-a908-466bd8af8b31)

We will make all web applications (i.e. Node-RED flow editor, Node-RED dashboard, ...) available on the default https port 443, because then we don't need to remember the port numbers of all our applications.  But that means that we need to use a sub-path for every web application, to tell the reverse proxy to which application (containing a web server) he needs to route the https requests.

For example:
1. Navigate to `https://your-virtual-hostname/dashboard` in the browser.
2. Since no port has been specified in this url, the browser will add the default https port 443.  So the browser will convert in the background the url to `https://your-virtual-hostname:443/dashboard`
3. The https request will arrive in the reverse proxy of your Tailscale agent, which is listening on port 443.
4. The reverse proxy will investigate the target url of the http request, and will detect that it contains the sub-path `/dashboard`.
5. When you have configured the reverse proxy to forward this sub-path to `http://localhost:1880/dashboard`, the https requests need to be converted to http requests.
6. Therefore the reverse proxy will do ***SSL termination***, and forward the http request to `http://localhost:1880/dashboard`.
7. Node-RED is listening to port 1880, and will detect the sub-path `/dashboard` in the target url of the http requests that arrive.
8. Based on that sub-path, the http request will be send to the dashboard application.
9. Finally you will see the dashboard in your browser, although you didn't have to specify any port numbers in the url.

So using sub-paths in urls can greatly simplify url's and hide multiple applications behind a single port.

From this we can conclude some rules of thumb:
+ When multiple applications are listening to the same (source of target) port, we need to use sub-paths to route the requests to a particular application.
+ When only a single application listens to a port, there is no need to use a sub-path.  Because all requests arriving on that port will be forwarded to that one application anyway.  However you might in this case also use sub-paths, to allow other applications to be added afterwards on the same port.  See for example port 8443 in the drawing, which uses a *"google_home"* sub-path, even though there is only one application listening.  That way you can afterwards easily use the same funnel for other third-party services, because currently Tailscale limits the number of ports that can be used for funnels.
+ When no sub-paths are being used, we say that the requests are being send to the root path `/`.

## Forwarding rules

The configuration of a reverse proy will contain a number of forward rules like this one (which we will setup later on via `tailscale funnel ...` and `tailscale serve ...` commands):
```
<source protocol> <source port> <source path> --> <target protocol>://<target port>:<destination path>
```
In other words: listen to the source port for some protocol (http or https), and url's containing some source path.  Then forwarding matching requests to the target url.

Note that you need to specify inside the applictions that they are running at a sub-path, so they can automatically add the sub-path to every incoming http request.  It won't work if the application doesn't support sub-paths.
