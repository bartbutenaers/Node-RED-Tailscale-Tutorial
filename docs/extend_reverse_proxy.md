# Extend the reverse proxy

In the previous modules of this tutorial, we have setup the reverse proxy in the Tailscale agent to do a couple of things:
+ Serve the Node-RED flow editor withing your tailnet (on port 443), which can be accessed like this:
   ```
   https://<your-virtual-hostname>/flow_editor
   ```
+ Make the node-red-contrib-google-smarthome node public available on the internet via a funnel, which can be accessed like this:
   ```
   https://<your-virtual-hostname>:8443/google_home
   ```

Now we are going to extend the reverse proxy a bit more.  We want all our local services (running on the Raspberry) to become available via the Tailscale agent, because that offers us some advantages:
+ All the services can be accessed via https, based on the same LetsEncrypt certificate.
+ All the services will be available within your tailnet.  That way you will be able to access them via your tailnet, which means even when you are not at home.
+ The url's are easy to remember, among others since you don't have to specify port numbers anymore.
+ ...

## Access the Node-RED dashboard
Similar to serving the flow editor, it is now time to serve the dashboard in the tailnet:
```
sudo tailscale serve --https=443 --bg --set-path /dashboard http://localhost:1880/dashboard
```
As a result, the dashboard will be accessible via:
```
https://<your-virtual-hostname>/dashboard
```

## Access other services
As describe above, it is useful to make other services availble in your tailnet.  

CAUTION: in order to be able to make a service accessible via a sub-path, the service should support custom base url's.  Similar to how we have changed before the Node-RED `httpAdminRoot` setting.

For example I can specify a custom base url for my VictoriaMetrics timeseries database, via a startup parameter:
```
victoria-metrics-prod -storageDataPath /var/lib/victoriametrics -retentionPeriod 100y -http.pathPrefix /victoria_metrics
```
That way it becomes possible to serve the VictoriaMetrics web interface, via that sub-path *"victoria_metrics"*:
```
sudo tailscale serve --https=443 --bg --set-path /victoria_metrics http://localhost:8428/victoria_metrics
```
Now the database will become available via the following url:
```
https://<your-virtual-hostname>/victoria_metrics
```

## The final result
Finally all our services are available (public or within our tailnet) via https, based on a LetsEncrypt certificate.  

Show the entire reverse proxy config via `sudo tailscale funnel status` or `sudo tailscale serve status`:
```
# Funnel on:
#     - https://<your-virtual-hostname>.<your-tailnet-name>.ts.net:8443

https://<your-virtual-hostname>.<your-tailnet-name>.ts.net (tailnet only)
|-- /dashboard        proxy http://localhost:1880/dashboard
|-- /flow_editor      proxy http://localhost:1880/flow_editor
|-- /victoria_metrics proxy http://localhost:8428/victoria_metrics

https://<your-virtual-hostname>.<your-tailnet-name>.ts.net:8443 (Funnel on)
|-- /google_home proxy http://localhost:3001
```
Which can be summarize like this:
![image](https://github.com/user-attachments/assets/872779fb-350a-4709-9361-76329c92ab8c)
