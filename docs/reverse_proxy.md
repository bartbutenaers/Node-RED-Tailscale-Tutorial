# Reverse proxy

A reverse proxy is a server/application/cloudservice that sits in front of web servers to intercept and inspect incoming client requests, before forwarding those requests to the web servers.  It can be used to:
+ Add some extra security to the web servers.
+ Add load balancing, i.e. distribute the client requests evenly across the web servers.
+ Forwarding requests via path-based routing.
+ Handle incoming https connections, and do SSL/TLS termination (i.e. decrypt the traffic and forward the requests as plain http).
+ Manage certificates, for example request and periodically renew LetsEncrypt certificates.
+ ...

The Tailscale agents also have an embedded reverse proxy, which we will use to handle https connections (via LetsEncrypt certificates) and path-based routing.

## Path-based routing

Path-based routing means that the reverse proxy looks at the URL inside the incoming http requests, to decide to which web server those requests need to be send.

Let's take a look at the path-based routing that will be constructed step-by-step, throughout the remaining chapters of this tutorial:

![image](https://github.com/user-attachments/assets/faf0f667-8b68-4d0d-a5cb-422b7c313c7b)

We will make all web applications (i.e. Node-RED flow editor, Node-RED dashboard, ...) available on the default https port 443.  But that means that we need to use sub-paths, to tell the reverse proxy to which web server he needs to route the https requests.

For example:
1. Navigate to `https://your-virtual-hostname/dashboard` in the browser.
2. Since no port has been specified in this url, the browser will add the default https port 443.  So the url will become `https://your-virtual-hostname:443/dashboard`
3. The https request will arrive in the reverse proxy of your Tailscale agent, which is listening on port 443.
4. The reverse proxy detects the sub-path `/dashboard`, so he will - after SSL termination - forward the http request to `localhost:1880/dashboard` (based on its configuration).
5. Node-RED is listening to port 1880, and will detect the sub-path `/dashboard` in the arriving http requests.
6. Based on that sub-path, the http request will be send to the dashboard application.

From this we can conclude some rules of thumb:
+ When multiple applications are listening to the same (source of target) port, we need to use sub-paths to route the requests to the corresponding application.
+ When only a single application listens to a port, there is no need to use a sub-path.  Bcause all requests on that port will be forwarded to that one application anyway.  However you might in this case also use sub-paths, to allow other applications to be added afterwards on the same port.  See for example port 8443 in the drawing, which will be used here for all (future) public funnels.
+ When no sub-paths are being used, we say that the requests are being send to the root path `/`.

## Summary

The configuration of a reverse proy will contain a number of forward rules like this one:
```
<source port>/<source path> --> forward --> <target port>/<destination path>
```
+ If there are multiple applications listening to the same source_port, each application will need to get a source_path.  Otherwise the source_path will be `/`.
+ If there are multiple applications listening to the same target_port, each application will need to get a target_path.  Otherwise the target_path will be `/`.

Note that you need to specify inside applictions that they are running at a sub-path, so they can automatically add the sub-path to every incoming http request.  It won't work if the application doesn't support sub-paths.
